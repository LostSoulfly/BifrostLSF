# Bifrost
Lightweight experimental cryptoprotocol :lock: :key:

## Disclaimer
I'm just an amateur who's interested in cryptography and networking. This protocol or its implementation may be heavily flawed, and I promise absolutely no expectation of security. If you're designing a security critical application, please consider using a mature and well-documented cryptoprotocol such as TLS. Thank you.

## What is Bifrost?
Bifrost is a cryptoprotocol, designed to be reliable, secure, lightweight and easy to understand. The whole library is around 1k lines of fully documented C#. Bifrost was designed in response to TLS, which has a very long and verbose specification document. In contrast, Bifrost is very easy to understand and doesn't require much effort to set up.

## Cryptographic primitives
Bifrost mostly depends on the excellent [BouncyCastle](http://bouncycastle.org/) library to do crypto. The used primitives are:

|Function|Primitive|
|---|---|
|Authentication|RSA-2048|
|Signatures|RSA-2048 with SHA256|
|Encryption|AES-256 in GCM|
|Key stretching|HKDF with SHA-512|
|Key exchange|ECDHE with the P-521 curve|

## Public key infrastructure
Since Bifrost is designed to be simple, it has its own PKI designed around PEM keypairs and raw signature files.

## Message format
Read more about Bifrost's message format in the [wiki](https://github.com/hexafluoride/Bifrost/wiki/Message-format).

## Simple example
Server side:

``` csharp
TcpListener listener = new TcpListener(8888);
listener.Start();
var client = listener.AcceptTcpClient();

TcpTunnel tunnel = new TcpTunnel(client);
ServerLink link = new ServerLink(tunnel);
link.LoadCertificatesFromFiles("test.ca", "server.privkey", "server.sign");

if(!link.PerformHandshake())
{
  Console.WriteLine("Handshake failed!");
  return;
}

link.OnDataReceived += (l, data) =>
{
  Console.WriteLine("Received {0} bytes from client: {1}", data.Length, Encoding.UTF8.GetString(data));
  l.Send(data);
};

link.StartThreads();

Console.ReadLine();
```

Client:
``` csharp
TcpClient client = new TcpClient("localhost", 8888);
TcpTunnel tunnel = new TcpTunnel(client);
ClientLink link = new ClientLink(tunnel);
link.LoadCertificatesFromFiles("test.ca", "client.privkey", "client.sign");

if(!link.PerformHandshake())
{
  Console.WriteLine("Handshake failed!");
  return;
}

link.OnDataReceived += (l, data) =>
{
  Console.WriteLine("Received {0} bytes from server: {1}", data.Length, Encoding.UTF8.GetString(data));
};

link.StartThreads();
link.Send(Encoding.UTF8.GetBytes("Hello World!"));

Console.ReadLine();
```
