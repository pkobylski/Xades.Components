# Xades.Components

Library and sample demo applications for creating XAdES signatures in C# (.NET Standard 2.0).

## Requirements
- .NET Standard 2.0 (compatible with .NET Framework 4.6.1+, .NET Core 2.0+, .NET 5+, and others).
- Valid X509 certificate (with private key) for signing.

## Features
- XAdES signatures: Enveloped, Enveloping, External, Internal.
- Hash algorithms: SHA1, SHA256.
- Generate a signature for an XML document or Base64 input data.

## Quick Start
```csharp
using System;
using System.Security.Cryptography.X509Certificates;
using System.Xml;
using Xades.Components.Xades; // przestrzeń nazw z XadesDocument

class Demo
{
    static void Main()
    {
        // Załaduj certyfikat (np. z magazynu lub pliku PFX)
        var cert = new X509Certificate2("sciezka-do-cert.pfx", "haslo");

        // Przygotuj dokument XML do podpisu
        var xml = new XmlDocument();
        xml.PreserveWhitespace = true; // zachowanie białych znaków jeśli wymagane
        xml.LoadXml("<Root>Test</Root>");

        // Utwórz obiekt podpisu (domyślnie SHA256 + Enveloped)
        var xades = new XadesDocument(CryptType.SHA256, EnvelopeType.Enveloped)
        {
            SerialNumber = "NUMER_SERYJNY_OPCJONALNY" // jeśli posiadasz licencję
        };

        // Podpisz dokument
        XmlDocument signed = xades.SignXades(xml, cert);

        // Wyświetl wynik
        Console.WriteLine(signed.OuterXml);
    }
}
```

## Base64 Data Signature
```csharp
XmlDocument signedXml = null;
string base64 = "..."; // dane wejściowe
var xades = new XadesDocument(CryptType.SHA256, EnvelopeType.Enveloping);
byte[] wynik = xades.SignXadesBase64File(base64, cert, ref signedXml, "plik.txt");
string podpisBase64 = Convert.ToBase64String(wynik);
```

## Licensing
Author of the signature components: Piotr Kobylski – contact: kobylski.piotr@gmail.com. For commercial use of the components, please notify the author.

## Support
For licensing or technical questions: kobylski.piotr@gmail.com.
