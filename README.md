# CryptLink.SigningFramework
A convenient signing and hashing framework, anything you can serialize, you can hash and sign. Allows for partial class hashing and custom implementations. 

Also provides a IComparable byte[] wrapper for efficient comparison (`Hash.Compare(byte[], byte[])`) and sorting and x509Certificate management features.

Nuget package: [CryptLink.SigningFramework](https://www.nuget.org/packages/CryptLink.SigningFramework/)

[![License: LGPL v3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
[![Build status](https://ci.appveyor.com/api/projects/status/j9iof6d4cb7kaqal/branch/master?svg=true)](https://ci.appveyor.com/project/CryptLink/signingframework/branch/master)
[![NuGet](https://img.shields.io/nuget/v/CryptLink.SigningFramework.svg)](https://www.nuget.org/packages/CryptLink.SigningFramework/)

## Examples
The signing framework strives to make common cryptography related tasks simple and extensible. We believe good security should be as simple as as their concepts, and extensible to any object.

### Simple Hashing Example
Hashing any object that implements IHashable:
``` C#
    var h1 = new HashableString("Test Value");
    h1.ComputeHash(HashProvider.SHA256);
```

### Custom Hashing Example
Below is a full example that defines a new partially hashed object, and a program that uses it.

``` C#
using System;
using CryptLink.SigningFramework;

namespace CryptLink.SigningFrameworkExamples
{
	/// <summary>A simple object that can be hashed</summary>
	[Serializable]
	public class HashableWidgetExample : Hashable {

		[HashProperty]
		public int ID { get; set; }
		
		[HashProperty]
		public string Name { get; set; }

		//A field that does not change the hash
		public int PurchaseCount { get; set; }
	}

    class Program {
        static void Main(string[] args) {
            var widget = new HashableWidgetExample() {
                ID = 1,
                Name = "Widget",
                PurchaseCount = 9001
            };

            widget.ComputeHash(HashProvider.SHA256);
            Console.WriteLine(widget.ComputedHash);
        }
    }
}
```

### Signing
Signing is overload of hashing, the only requirement is a valid x509 certificate with a private key.

``` C#
using (X509Store store = new X509Store(StoreName.My, StoreLocation.LocalMachine)) {
    store.Open(OpenFlags.ReadOnly);
    X509Certificate2Collection certCol = store.Certificates.Find(X509FindType.FindBySerialNumber, "123456", true);
    var cert = new Cert(certCol[0]);

    widget.ComputeHash(HashProvider.SHA256, cert);
    widget.Verify(cert);
}
```

### Custom Hashing
If the type you want to hash is not serializable, or want to serialize the binary data in a specific way, you can override `GetHashableData()`

``` C#
public new byte[] GetHashableData() {
	//Return the hashable bytes here
}
```

### Comparing byte[]
Dotnet does not have a native way to compare the contents of two byte[] array contents, but the `ComparableBytes` does:

``` C#
    byte[] bytesA = { 1, 2, 3 };
    byte[] bytesB = { 1, 2, 3 };

    // Standard compare (by reference)
    if (bytesA == bytesB) {
        // Evaluates as false (A and B are different objects)
        Assert.Fail();
    }

    // using CryptLink.SigningFramework;
    if (bytesA.ToComparable() == bytesB.ToComparable()) {
        // Evaluates as true (A and B have the same byte values)
    }

    // Using the ComparableBytes Wrapper
    var cBytesA = new ComparableBytes(bytesA);
    var cBytesB = new ComparableBytes(bytesB);

    if (cBytesA == cBytesB) {
        // Evaluates as true (A and B have the same byte values)
    }
```

## Features / Classes
This library implements a number of classes, abstractions and interfaces, this list is in order of abstraction:

### IComparable
The standard class for implementing any comparable (and sortable) object, provided by dotnet standard.

### CompareableBytesAbstract
CryptLink's lowest level abstract implementation of a comparable array of bytes. Contains base functionality and IComparable implementation. Not intended to be used directly.

### CompareableBytes
A minimal class for implementing `ComparableBytesAbstract`. Lowest level type intended direct use to storing bytes that need to be compared.

### Hash
Implements `ComparableBytes` and adds features for computing the hash for arbitrary bytes. Intended for storing all hashes/signatures.

### IHashable
Interface that defines the basic properties and functions needed to hash an object.

### Hashable
Abstract that implements higher level functionality for computing and verifying hashes. Intended to be inherited by any object that wants to implementing hashing.

## Implementations
Some implementations of `Hashable` for utility, reference and convenience:

### HashableString
A sample implementation of `Hashable` that makes strings hashable and signable, useful for testing.

### Cert
A wrapper for X509Certificate2 that implements `Hashable` and OS independent storage and protection.
