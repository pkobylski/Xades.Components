# Xades.Components

.NET library for creating **XAdES-BES** XML signatures (Enveloped, Enveloping, detached) with an X.509 certificate.
Built for Polish e-government gateways such as e-Deklaracje, KSeF and PUESC.

```
dotnet add package Xades.Components
```

## Requirements

| Platform | Minimum | Tested on |
|---|---|---|
| .NET Framework | 4.6.2 (4.7.2+ recommended) | 4.8 |
| .NET | 8.0 | 10.0 |

The library targets `netstandard2.0`; the minimum versions above come from its dependency
`System.Security.Cryptography.Xml`, which supports only these platforms.

Certificate: X.509 with an **RSA** private key. Keys in a classic CSP and in a CNG **KSP** (Key Storage Provider),
including smart cards and tokens, are supported (since 2.0.3).

## Quick start

```csharp
using System;
using System.Security.Cryptography.X509Certificates;
using System.Xml;
using Xades.Components;        // CryptType, EnvelopeType
using Xades.Components.Xades;  // XadesDocument

// Certificate with a private key: from a PFX file or from the Windows certificate store.
// On .NET 9+ prefer X509CertificateLoader.LoadPkcs12FromFile(path, password).
var cert = new X509Certificate2("certificate.pfx", "password");

var xml = new XmlDocument { PreserveWhitespace = true };
xml.Load("document.xml");

var xades = new XadesDocument(CryptType.SHA256, EnvelopeType.Enveloped)
{
    SerialNumber = "YOUR-SERIAL-NUMBER" // see Licensing
};

XmlDocument signed = xades.SignXades(xml, cert);
signed.Save("document.xades.xml");
```

`SignXades` returns a **new** document; the input `XmlDocument` is not modified.

## Supported modes

| `EnvelopeType` | Input | SHA-1 | SHA-256 | Result |
|---|---|---|---|---|
| `Enveloped` | `XmlDocument` (`SignXades`) | ✔ | ✔ | the input document with `ds:Signature` appended to the root element |
| `Enveloped` | Base64 data (`SignXadesBase64File`) | ✔ | ✔ | `<body>` containing `ds:Signature` with the data in `ds:Object` |
| `Enveloping` | `XmlDocument` (`SignXades`) | – | ✔ | `ds:Signature` as the root element with the document inside `ds:Object` |
| `Internal` | Base64 data (`SignXadesBase64File`) | ✔ | ✔ | `ds:Signature` with the data in `ds:Object` |
| `External` | Base64 data (`SignXadesBase64File`) | ✔ | ✔ | detached `ds:Signature`; the reference URI is the `fileToSignName` argument |

Combinations not listed (e.g. `Enveloping` with SHA-1 or with Base64 input) are not supported and throw an exception.

Signature details: RSA-SHA1 / RSA-SHA256 (`http://www.w3.org/2001/04/xmldsig-more#rsa-sha256`),
exclusive canonicalization, XAdES 1.3.2 `SignedProperties` with `SigningTime` and `SigningCertificate`.

### Signing Base64 data

```csharp
string base64 = Convert.ToBase64String(File.ReadAllBytes("invoice.pdf"));

var xades = new XadesDocument(CryptType.SHA256, EnvelopeType.External);
XmlDocument signedXml = null;
byte[] signature = xades.SignXadesBase64File(base64, cert, ref signedXml, "invoice.pdf");

File.WriteAllBytes("invoice.pdf.xades", signature);
```

## Behaviour worth knowing (Enveloped)

- **Comments are removed** from the signed document. XML signatures never cover comments, and leaving them in
  made some verifiers reject the signature.
- **Whitespace is preserved.** Load the input with `PreserveWhitespace = true` if its formatting matters to you;
  the returned document can be saved with `Save()` without breaking the signature.

## Licensing

The library is **not open source**. Without a valid `SerialNumber`:

- every signature writes a notice to standard error,
- only **4 signatures per process** are allowed; the 5th call throws `InvalidOperationException`.

An invalid `SerialNumber` throws an exception. To obtain a serial number, contact the author.

## Changelog

**2.0.3**
- Certificates with keys in a CNG KSP no longer fail with "Invalid provider type specified".
- Enveloped: comments in the input document and `XmlDocument.Save()` no longer invalidate the signature;
  the input document is no longer modified.
- Signatures produced by the library can be verified with `SignedXml.CheckSignature()` on .NET 8+.

## Contact

Piotr Kobylski – kobylski.piotr@gmail.com (licensing and technical support)
