# Serialization

## Table of Contents

- [Content](#content)

## Content
Serialization is a method to persist the state of an object in order to have the ability to recreate the same object when required. When the object is required to travel electronically over the network, in such cases the objects are serialized at one end and deserialized at the other end. Serialization is one of the fundamental requirements for techniques such as .Net Remoting.

Serialization in .Net are of four types as shown in:
/

1. XML serialization allows you to serialize objects into XML data format. However it can convert only public properties of an object into xml data. Therefore it is referred to as shallow serialization.
1. In SOAP and BINARY serialization technique the state of the entire object is serialized into a stream of bytes. In cases where the object contains a reference to other objects, even those are serialized. This type of serialization is known as deep serialization. This overcomes the disadvantage of XML serializer to serialize the private fields of an object.

The .Net Framework also provides the SoapFormatter and the BinaryFormatter classes. These classes implement the IRemotingFormatter and the IFormatter interfaces.

The main methods which encapsulate the functionality of serialization are the Serialize and Deserialize methods.

Here we study the soap formatter.

[Serializable]
public class secreteClass
{
private int myaccountnumber = 0;
[NonSerialized()]
private string password = "qwerty321";

public int MyAccNum
{
get
{
return myaccountnumber;
}
set
{
myaccountnumber = value;
}
}
public string Passwd
{
get
{
return password;
}
set
{
password = value;
}
}
public secreteClass()
{

}
}

Note: Here password is not serialized and hence it is shown as [NonSerialized()].

using System.IO;
using System.Collections;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters.Soap;
Public static void Main()
{
try
{

secreteClass objSecrete = new secreteClass();
objSecrete.MyAccNum = 123321456;

//Soap serialization technique
SoapFormatter formatter = new SoapFormatter();
Stream objfilestream = new FileStream("c:\\Myserialzed.xml", FileMode.Create, FileAccess.Write,FileShare.None);

formatter.Serialize(objfilestream, objSecrete);
objfilestream.Close();
//deserialization
Stream objreadstream = new FileStream("c:\\Myserialzed.xml", FileMode.Open, FileAccess.Read,FileShare.Read);
secreteClass objSecrete2 = (secreteClass)formatter.Deserialize(objreadstream);

int Myaccno = objSecrete2.MyAccNum;
Console.Writeln("AccNo:{0} " + Myaccno.ToString());
}
catch (Exception ex)
{
}
}

Note: When building this code, you must reference the System.Runtime.Serialization.Formatters.Soap.dll assembly.

This is the XML file(Myserialized.xml) screenshot, where the data is serialized and stored.

Myserialized.xml

<SOAP-ENV:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:clr="http://schemas.microsoft.com/soap/encoding/clr/1.0" SOAP-ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
<SOAP-ENV:Body>
<a1:serderlize_x002B_secreteClass id="ref-1" xmlns:a1="http://schemas.microsoft.com/clr/assem/App_Web_1imlsxqk%2C%20Version%3D0.0.0.0%2C%20Culture%3Dneutral%2C%20PublicKeyToken%3Dnull">
<myaccountnumber>123321456</myaccountnumber>
</a1:serderlize_x002B_secreteClass>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>



[↑ Back to Top](#table-of-contents)

[↑ Back to Top](#table-of-contents)
