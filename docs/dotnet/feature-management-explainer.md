# Feature Management in CAT.Portal

## Table of Contents

1. [What Feature Management Means](#what-feature-management-means)
2. [How It Is Implemented in This Repository](#how-it-is-implemented-in-this-repository)
3. [What Configuration Drives It](#what-configuration-drives-it)
4. [Concrete UI Flag Example](#concrete-ui-flag-example)
5. [End-to-End Flow for the Example Flag](#end-to-end-flow-for-the-example-flag)
6. [Where the UI Changes](#where-the-ui-changes)
7. [What Controls Refresh Behavior](#what-controls-refresh-behavior)
8. [Will a Toggle Immediately Reflect on the UI?](#will-a-toggle-immediately-reflect-on-the-ui)
9. [Short Summary](#short-summary)

## What Feature Management Means

Feature Management is a way to turn application behavior on or off without changing code.

In simple language, a feature flag is a switch. Instead of hardcoding "always use the new behavior" or "always use the old behavior," the application asks a question at runtime:

- Is this feature enabled right now?

Based on that answer, the app decides which behavior, page, route, or UI element to use.

This is useful for:

- Gradually rolling out new features
- Turning a feature off quickly if something goes wrong
- Testing new behavior in one environment before enabling it everywhere
- Avoiding redeploys just to change a runtime switch

## How It Is Implemented in This Repository

In this repository, the feature flags are not stored in code as true/false values. Code stores the flag names and checks them. The actual current values come from Azure App Configuration.

### Main backend wiring

The main Web API host connects to Azure App Configuration in `src/CAT.Portal.WebApi/Catalyst.Core/Configuration/AzureAppConfig/AzureAppConfigurationBuilder.cs`.

That builder:

- Connects to Azure App Configuration
- Loads configuration for the current environment label
- Enables feature flags
- Configures refresh behavior
- Registers a sentinel key for broader config refresh

The main host then registers feature management in `src/CAT.Portal.WebApi/Catalyst.Web.Host/Startup/Startup.cs`:

- `services.AddFeatureManagement()`
- `services.AddAzureAppConfiguration()`
- `app.UseAzureAppConfiguration()`

That means the backend is wired to evaluate feature flags through `IFeatureManager`.

### Backend API that exposes enabled flags

There is already an app service in `src/CAT.Portal.WebApi/Catalyst.Application/Configuration/FeatureManagement/FeatureManagementAppService.cs`.

It does this:

- Asks `IFeatureManager` for all known feature flag names
- Checks which ones are enabled
- Returns only the enabled names

The method is `GetEnabledFeatures()`.

### Angular frontend wiring

The Angular app uses `src/CAT.Portal.Web/src/shared/feature-management/feature-manager.service.ts`.

That service:

- Calls the backend endpoint through `FeatureManagementServiceProxy`
- Retrieves the enabled feature names from `/api/services/app/FeatureManagement/GetEnabledFeatures`
- Stores those enabled names in memory
- Lets UI code ask `featureOn(featureName)`

The generated client route lives in `src/CAT.Portal.Web/src/shared/service-proxies/service-proxies.ts`.

### Angular startup behavior

The Angular app loads feature flags during startup using `APP_INITIALIZER` in `src/CAT.Portal.Web/src/root.module.ts`.

During app startup:

- The app initializes session state
- It resolves `FeatureManagerService`
- It calls `featureManagerService.initialize()`
- That fetches the enabled feature list before the app finishes booting

This means Angular pages, guards, and components can immediately ask whether a feature is enabled.

### Where feature names live in Angular

Frontend feature flag names are centralized in `src/CAT.Portal.Web/src/shared/feature-management/feature-flags.ts`.

Examples include:

- `SubmitterProductDetails.EnableNewView`
- `ManageProductFamilies.Portal`
- `SupplierProduct.UIEnrichment`
- `CAT.Portal.EnableSignalR`

## What Configuration Drives It

The behavior is controlled by configuration, not by changing code.

### Main control points

There are four main things that drive feature management:

1. Azure App Configuration connection string
2. App name used for namespacing and sentinel keys
3. Environment label used to choose the right values
4. Refresh intervals that determine how often values are reloaded

### Main Web API configuration

In `src/CAT.Portal.WebApi/Catalyst.Web.Host/appsettings.json`, the important settings are under `FoundationConfigurationOptions`:

- `ConnectionString`
- `AppName = CatPortalWebApi`
- `AppConfigRefreshIntervalInMinutes = 5`
- `FeatureFlagRefreshIntervalInSeconds = 360`

### What those settings mean

- `ConnectionString`: tells the app which Azure App Configuration store to read from
- `AppName`: helps build the sentinel key, such as `CatPortalWebApi:Sentinel`
- `AppConfigRefreshIntervalInMinutes`: controls how often normal configuration refresh is checked
- `FeatureFlagRefreshIntervalInSeconds`: controls how often feature flags are refreshed

### Environment labels

In `src/CAT.Portal.WebApi/Catalyst.Core/Configuration/AzureAppConfig/AzureAppConfigurationBuilder.cs`, the app loads:

- Unlabeled settings
- Settings labeled for the current environment

That means Dev, QA, and Prod can all have different feature flag values.

## Concrete UI Flag Example

A good UI-facing example is:

- `SubmitterProductDetails.EnableNewView`

In Angular, this is defined in `src/CAT.Portal.Web/src/shared/feature-management/feature-flags.ts` as:

- `FeatureFlags.SubmitterProductDetailsEnableNewView = "SubmitterProductDetails.EnableNewView"`

In plain language, this flag means:

- Should the app use the new product-details experience, or keep using the old one?

## End-to-End Flow for the Example Flag

Here is the full path from Azure to the Angular UI.

### 1. Azure App Configuration

A feature flag named `SubmitterProductDetails.EnableNewView` is created or updated in Azure App Configuration.

### 2. Backend host loads feature flags

The portal host connects to Azure App Configuration through `src/CAT.Portal.WebApi/Catalyst.Core/Configuration/AzureAppConfig/AzureAppConfigurationBuilder.cs`.

The host wiring in `src/CAT.Portal.WebApi/Catalyst.Web.Host/Startup/Startup.cs` registers the feature-management services and refresh middleware.

### 3. Backend evaluates enabled features

`src/CAT.Portal.WebApi/Catalyst.Application/Configuration/FeatureManagement/FeatureManagementAppService.cs` uses `IFeatureManager` to determine which flags are enabled.

It returns the enabled feature names from:

- `/api/services/app/FeatureManagement/GetEnabledFeatures`

### 4. Angular client calls the backend

The generated proxy in `src/CAT.Portal.Web/src/shared/service-proxies/service-proxies.ts` calls:

- `/api/services/app/FeatureManagement/GetEnabledFeatures`

### 5. Angular startup caches the enabled flags

During Angular startup, `src/CAT.Portal.Web/src/root.module.ts` calls `FeatureManagerService.initialize()`.

That service, in `src/CAT.Portal.Web/src/shared/feature-management/feature-manager.service.ts`, stores the enabled feature names in `_featureFlags`.

### 6. UI code asks whether the flag is on

When UI code needs to decide what to show, it calls:

- `featureManager.featureOn(FeatureFlags.SubmitterProductDetailsEnableNewView)`

If the flag name is in the enabled list, Angular treats the feature as on.

## Where the UI Changes

This example flag affects routing and UI behavior in multiple places.

### Route guard

In `src/CAT.Portal.Web/src/app/products/product-details-v2-can-match.guard.ts`, Angular only allows the V2 route when the flag is enabled.

That is the clearest page-level decision point.

### Seller product grid behavior

In `src/CAT.Portal.Web/src/app/products/seller-product-grid/seller-product-grid.component.ts`, the component sets:

- `isSubmitterProductDetailsNewViewEnabled = this.featureManagerService.featureOn(FeatureFlags.SubmitterProductDetailsEnableNewView)`

That lets the grid react differently when the new product-details view is active.

### Product mapping effect behavior

In `src/CAT.Portal.Web/src/app/products/product-details/product-mapping/effects.ts`, there is logic that only shows the old success notification when the new-view flag is off.

So this flag does not only affect routing. It also affects behavior inside the page flow.

## What Controls Refresh Behavior

For this flag to change without redeploy, three things need to work together.

### 1. Azure App Configuration must have the updated value

If Azure still says the flag is off, the app will keep using the old behavior.

### 2. The backend host must refresh feature flags at runtime

In the main portal host, this is wired by:

- `services.AddFeatureManagement()`
- `services.AddAzureAppConfiguration()`
- `app.UseAzureAppConfiguration()`

All of that is in `src/CAT.Portal.WebApi/Catalyst.Web.Host/Startup/Startup.cs`.

### 3. The Angular app must fetch the updated enabled-feature list

Angular currently loads feature flags during startup and stores them in memory. That means the browser session uses the cached list it fetched at initialization time.

So there are two refresh scopes to keep in mind:

- Server-side refresh: controlled by Azure App Configuration refresh intervals and middleware
- Client-side refresh: controlled by when Angular calls `GetEnabledFeatures()` again

This distinction matters. Even if the backend sees a new flag value, the Angular SPA may still need a page reload or a fresh feature fetch to pick up the latest enabled-feature list.

## Will a Toggle Immediately Reflect on the UI?

Not immediately, at least not with the current Angular UI flow.

There are two separate refresh hops:

1. Azure App Configuration to the backend
2. Backend to the browser UI

### Backend refresh timing

The main portal host is wired to refresh feature flags at runtime through `src/CAT.Portal.WebApi/Catalyst.Web.Host/Startup/Startup.cs` and `src/CAT.Portal.WebApi/Catalyst.Core/Configuration/AzureAppConfig/AzureAppConfigurationBuilder.cs`.

That means the backend is intended to pick up changed feature flag values without redeploying. But it is still not instant.

In `src/CAT.Portal.WebApi/Catalyst.Web.Host/appsettings.json`, the feature flag cache interval is:

- `FeatureFlagRefreshIntervalInSeconds = 360`

So the backend may take up to about 6 minutes to observe the new flag value.

### Frontend refresh timing

The Angular UI does not appear to subscribe to live feature updates.

Instead, it fetches the enabled-feature list during startup in `src/CAT.Portal.Web/src/root.module.ts`, which calls `FeatureManagerService.initialize()`.

That service in `src/CAT.Portal.Web/src/shared/feature-management/feature-manager.service.ts`:

- Calls the backend once
- Stores the enabled feature names in memory
- Serves later checks from that in-memory list

That means an already-open browser session will usually keep using the old feature state until the app reloads or explicitly fetches the feature list again.

### Practical answer

If you toggle a feature flag in Azure App Configuration:

- It will usually not reflect immediately on the UI
- The backend should see it after its refresh interval
- The frontend should see it after the browser reloads and fetches the enabled-feature list again

If refresh wiring is incomplete or broken in a service, the change may not reflect until redeploy. That is the specific problem the feature-flag refresh plan is intended to fix.

### How the updated value propagates when it works

When propagation works correctly, the flow is:

1. Azure App Configuration stores the new feature flag value.
2. The backend refresh middleware updates the server-side feature view after the configured cache interval.
3. `src/CAT.Portal.WebApi/Catalyst.Application/Configuration/FeatureManagement/FeatureManagementAppService.cs` returns the updated enabled-feature list.
4. Angular calls `/api/services/app/FeatureManagement/GetEnabledFeatures` through the generated client in `src/CAT.Portal.Web/src/shared/service-proxies/service-proxies.ts`.
5. `src/CAT.Portal.Web/src/shared/feature-management/feature-manager.service.ts` caches the new list.
6. Guards and components read the updated flag state and render the new UI path.

### Key distinction

The backend is designed for runtime refresh, but the frontend is effectively startup-cached.

So if the goal is near-immediate UI reflection after an Azure toggle, the frontend needs an explicit refresh mechanism instead of relying only on app-start initialization.

## Short Summary

For a UI flag such as `SubmitterProductDetails.EnableNewView`, the flow is:

1. Azure App Configuration stores the flag value.
2. The Web API host loads and refreshes feature flags through Azure App Configuration.
3. `FeatureManagementAppService.GetEnabledFeatures()` returns the enabled flags.
4. Angular calls that endpoint during startup.
5. `FeatureManagerService` caches the enabled names.
6. Angular guards and components check `featureOn(...)` and decide what UI to show.

For this example, if the flag is enabled, the new product-details UI path is allowed. If it is disabled, the app stays on the old behavior.
