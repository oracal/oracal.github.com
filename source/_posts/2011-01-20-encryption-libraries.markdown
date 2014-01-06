---
comments: true
date: 2011-01-20 20:44:29
layout: post
slug: encryption-libraries
title: Encryption Libraries
categories:
- c++
- encryption
- qt
---

Recently I was working on a small personal project and I was in need of a quick and easy encryption library that would be able to do quite a few different encryption algorithms. My only real prerequisites were that the library had to be in c++ (well c would do but I'd prefer it to be in oop form), the encryption algorithms had to include AES 128 and it had to be cross platform. I was quite surprised at how few encryption libraries there were for c++. The three mainstream cross platform libraries that I found were crypto++, Botan and QCA (Qt Cryptology Architecture) (there was also the OpenSSL library but that is written in c).

<!-- more -->I first looked at QCA considering my project was using Qt and since it uses a Qt-style API and Qt datatypes it was a very obvious first choice. I was quite surprised at how simple it was to perform an encryption, decryption cipher using AES 128, but I did have quite a bit of trouble using the QCA plugin with Qt Creator on my Windows PC, where as with Linux it was working within minutes. This is the code I had to get it working on linux, no guarantees for windows.

{% codeblock lang:cpp %}
// setup the encryption library
QCA::Initializer init;
// define the message to be encrypted
QCA::SecureArray arg = "Text to be encrypted";
// create a random 16 bit key
QCA::SymmetricKey key(16);
// create a random 16 bit initialisation vector
QCA::InitializationVector iv(16);

// check the encryption algorithm is supported
if(!QCA::isSupported("aes128-cbc-pkcs7"))
	qDebug()<<"AES128-CBC not supported!");
else
{
	// create the cipher defintion
  QCA::Cipher cipher(
          QString("aes256"),
          QCA::Cipher::CBC,
          QCA::Cipher::DefaultPadding,
          QCA::Encode, key, iv);

	// start encryption process
	QCA::SecureArray u = cipher.update(arg);

	QCA::SecureArray f = cipher.final();

	QCA::SecureArray cipherText = u.append(f);
	// end encryption process

	// convert to QString
	QString out = QCA::arrayToHex(cipherText.toByteArray());
}
{% endcodeblock %}

Since the QCA plugin was causing me trouble I thought I'd try out the other libraries. Crypto++ was the next obvious choice since it has fips accreditation on a lot of its algorithms. It seemed like a really good library with a lot of amazing features. However, I really struggled with its complex code and lack of understandable documentation. It had a very steep learning curve and was definitely not useful for a relative beginner into cryptography. I gave up trying to get it to work and tried my third library: Botan, and I was very glad I did. Here's a sample of the code needed to get it woorking but I really struggled with it.

{% codeblock lang:cpp %}

// key and IV setup
byte key[ CryptoPP::AES::DEFAULT_KEYLENGTH ], iv[ CryptoPP::AES::BLOCKSIZE ];
memset( key, 0x00, CryptoPP::AES::DEFAULT_KEYLENGTH );
memset( iv, 0x00, CryptoPP::AES::BLOCKSIZE );

// string and output setup
std::string plaintext = "Text to be encrypted";
std::string ciphertext;

// create cipher Text
CryptoPP::AES::Encryption aesEncryption(key, CryptoPP::AES::DEFAULT_KEYLENGTH);
CryptoPP::CBC_Mode_ExternalCipher::Encryption cbcEncryption( aesEncryption, iv );

CryptoPP::StreamTransformationFilter stfEncryptor(
        cbcEncryption,
        new CryptoPP::StringSink( ciphertext ) );

stfEncryptor.Put(
        reinterpret_cast<const unsigned char*> ( plaintext.data() ),
        plaintext.length() + 1 );

stfEncryptor.MessageEnd();

// convert to QString
QString output = QString(ciphertext.c_str())
{% endcodeblock %}

Now Botan by comparison was very easy to pick up. It uses a very nice programming paradigm using real life metaphors, such as pipes, filters and forks. Pipes are just conduits of data where processes are applied to the data flowing through, such as encrpytion, hashes etc. In Botan terminology these processes are called filters and a pipe can have as many or as little as you like. Forks are where you split the data so that you end up with different outputs using the same data, assuming the filters you use are different. The documentation is also very easy to comprehend. The only downside is that it is not accredited, but it is still a very well respected encryption library none the less.

{% codeblock lang:cpp %}

// initialise the Botan library
Botan::LibraryInitializer init;

Botan::RandomNumberGenerator *rnd = new Botan::AutoSeeded_RNG();
// generate random 16 bit key
Botan::SymmetricKey key(*rnd,16);
delete rnd;
// generate random 16 bit initialisation vector
Botan::SecureVector<Botan::byte> raw_iv(16);
Botan::InitializationVector iv(raw_iv, 16);
// define text to be encrypted
std::string str = "Text to be encrypted";

// generate cipher "pipe"
Botan::Pipe pipe(get_cipher("AES-128/CBC", key, iv, Botan::ENCRYPTION));

// add text to pipe
pipe.process_msg(str);

// get std:string encrypted output
std::string output = pipe.read_all_as_string();

// convert to Qstring
QString qoutput = QString(output.c_str());

{% endcodeblock %}

In conclusion if you're looking for a quick and cheerful encrpytion library thats easy to pick up, I'd say go for Botan. Now dependent on the program you're building and how much accreditation you need I'd say you might want to look at some of the others, but it might be worth trying Botan first to quickly create an early, perfectly function version of your program.
