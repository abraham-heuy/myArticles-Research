# TITLE: The Pillars of Communication(Encryption explained).

## Abstract: 
This article explores the fundamental principles of end-to-end encryption(E2EE) and its critical
role in securing modern digital communication. We begin by introducing the basic concept of encryption through the classic examples of Alice and Bob. The discussion then contrasts traditional encryption models with true E2EE, highlighting the enhanced security the latter provides. Finally, we address a complex, practical challenge: how to store encrypted messages in a database without compromising the privacy guarantees of E2EE.

## 1. Introduction: What is encryption?
def: Encryption is the process of converting information or data (plaintext) into a secret code (ciphertext) to prevent unauthorized access.
       - This mathematical process ensures that even if a message is intercepted, it remains unreadable to anyone except the intended recipient, who possesses the unique ability to decrypt it back into its original form.

       Scenario: let's meet Alice and Bob.
       Imagine Alice wants to send a private message, "Hello Bob," across an insecure network. If she sends it as plaintext, anyone, the service provider,could read it. To prevent this, Alice and Bob use encryption. 
       (i). The Goal: Alice wants to ensure that only Bob can read the message "Hello Bob."

      (ii). The Method: Alice uses a cryptographic "key" (a secret piece of information) to scramble her message. This process turns the readable plaintext into unreadable gibberish, or ciphertext (e.g., "62jcksHtsje7") .

     (iii). The Transmission: Alice sends this ciphertext over the network. If an eavesdropper intercepts it, they only see the gibberish.

     (iv). The Reception: When Bob receives the ciphertext, he uses his own key to unscramble (decrypt) it, turning it back into the original message, "Hello Bob"

## 2. Securing the Conversation: How E2EE Works?

Encryption can be applied in different ways. The most common distinction is between data "in transit" (moving across a network) and data "at rest" (stored on a device) . End-to-end encryption is the gold standard for securing data in transit.

End-to-end encryption (E2EE) is a system where only the communicating users can read the messages. Crucially, it prevents potential eavesdroppers, including telecom providers, Internet service providers, and even the provider of the communication service,from being able to access the cryptographic keys needed to decrypt the conversation.

This is different from transport-layer encryption (like the HTTPS used for secure websites), which only protects data as it travels from your device to the service's server. At that server, the data is decrypted, meaning the service provider could potentially see or store your information . With E2EE, the service provider's server only sees the encrypted, unreadable ciphertext.

Scenario: We use Alice and Bob
Key Generation: Alice and Bob each generate their own pair of keys. This pair consists of a public key and a private key (or secret key) . While the name can be confusing, the public key is shared openly, and the private key is kept absolutely secret .

  (i). Public Key Exchange: Alice sends her public key to Bob, and Bob sends his public key to Alice. This exchange can happen over an insecure channel, as public keys are designed to be shared.

  (ii). Encryption: When Alice wants to send a private message to Bob, she uses Bob's public key to encrypt it. Once encrypted with Bob's public key, the message can only be decrypted with Bob's corresponding private key.

  (iii). Secure Transmission: Alice sends the encrypted message (ciphertext). The server relays this ciphertext to Bob. Even if the server is compromised, the attacker sees only the ciphertext.

  (iv). Decryption: Bob receives the ciphertext and uses his private key to decrypt it, revealing the original message.

This system ensures that no middleman, not even the server facilitating the chat,can ever read the contents of Alice and Bob's conversation.

In practice, E2EE systems often use a more efficient method called hybrid encryption due to the speed limitations of public-key cryptography. For instance, when Alice sends a long message, her app might create a temporary, random key for symmetric encryption (like AES). She then encrypts her long message with this fast, symmetric "session key." Finally, she protects that session key by encrypting it with Bob's public key. Bob decrypts the session key with his private key, then uses it to decrypt the message .

A famous example of an E2EE protocol is Signal Protocol, known for its strong security properties like forward secrecy, which ensures that messages remain safe even if long-term private keys are compromised in the future.

## 3. The Persistent Challenge: Encrypting Messages in a Database
While securing messages in transit is critical, they eventually need to be stored. The question then becomes: How  can we store these messages in a database while maintaining the privacy guarantees of E2EE?  After all, the database is often controlled by the service provider, who we have just ensured cannot read the messages. 

The core principle is that the database should store the message in its ciphertext form, as it appeared during transmission. The service provider's server should never have access to the decryption keys. Therefore, when Bob's device requests his message history, the server simply retrieves the already encrypted message from the database and sends it to Bob. Bob's client app then uses his private key to decrypt it locally.

However, storing messages this way presents a significant functional challenge: Searchability.
i.e: Have a database with millions of encrypted message. If Alice wants to find an old message containing the word "diner", the server cannot simply seach for "diner" (because all the messages are encrypted remember!). The server would have to download every sincle message to Bob's device(this is a privacy and security risk), or Bob's device would have to download and decrypt its entire message history locally(massive bandwidth & computation problem).

This creates a need for specialized techniwues to protect data at rest in a way that allows for features/functionalities like search. 

### The breakthrough: Compact Key Storage(CKS). 
 One cutting-edge approach to managing the keys needed for this data, formalized by researchers including those from NYU and Zoom, is called Compact Key Storage (CKS) . This is particularly relevant for cloud backups of E2EE conversations.

 The problem with a simple backup is that it often undermines advanced security features like forward secrecy. If you back up all your messages with a single, unchanging key, and that key is later stolen, all your past messages are exposed.

CKS offers a "right" solution to this problem by allowing a set of participants to securely delegate the storage of an ever-increasing set of keys to a server, while each client maintains only a small, manageable state

Here’s how it helps in practice:

   (i). Maintaining Security: Clients can update their state as they learn new keys (maintaining post-compromise security) or "forget" old keys (achieving forward secrecy) without needing to constantly interact with the server .

   (ii). Efficient Delegation: When a new user, say Charlie, joins a group chat with Alice and Bob, he needs access to the history of messages. With CKS, access to the entire history of keys can be efficiently delegated to Charlie, who can then share the same compact storage on the server. This avoids the wasteful and insecure practice of re-encrypting the entire message history for the new member.

### Searching Encrypted Data: Deterministic Encryption
Another practical need is the ability to search for specific records without decrypting them, a common requirement for data like email subject lines or identifiers. A solution for this involves a form of deterministic encryption .

The goal is to encrypt sensitive information, like a social security number, so that:

    1. An attacker who steals the database cannot read it.

    2. The application can still perform exact-match searches (e.g., "find the record for SSN 123-45-6789").

This is achieved by encrypting the data in a way that the same plaintext (e.g., the same SSN) always produces the same ciphertext. This allows the database to index and search for that specific encrypted value .

However, this method has a critical vulnerability: it is susceptible to frequency analysis. If an attacker knows the format of the data (e.g., all possible SSNs), they could theoretically build a "rainbow table" by encrypting every possible SSN with the compromised key and matching the results to the database. Therefore, the security of this method relies entirely on keeping the encryption key absolutely secret.

## 4. Case Study: Meta app => eg Whatsapp:
To see how E2EE works in the real world, let's look at WhatsApp. It's a perfect example because it uses the Signal Protocol, widely considered one of the most secure messaging protocols in the world . The magic is that it provides this high level of security automatically for over a billion users .

Scenario: Alice and Bob communicating using WhatsApp.
   1. The Setup (One Time Only): When Alice first installs WhatsApp on her phone, the app automatically creates two special keys for her:

       - A Public Key, which is like a lock that anyone can use to secure a message for her.

       - A Private Key, which is like her unique key to open those locks. This key never leaves her phone . Her public keys are automatically uploaded to WhatsApp's servers .

   2. Starting a Chat: When Alice wants to message Bob, her phone asks WhatsApp's server for Bob's public keys.

   3. Sending a Message (The Lock): Alice types "Hi Bob!". Her phone takes Bob's public lock and uses it to scramble the message. It instantly turns "Hi Bob!" into a complex, unreadable code (ciphertext) . Crucially, this happens on her phone before the message is even sent. Even WhatsApp's servers, which relay the message, only see the scrambled code .

   4. Receiving and Reading (The Key): Bob's phone receives this scrambled code from the server. His phone then automatically uses his own private key(the one that never left his device) to unscramble it, turning it back into the plain English "Hi Bob!" that he can read .
Because of this system, even if a hacker intercepted the message or WhatsApp's servers were compromised, they would only find the scrambled, unreadable code . The conversation remains a private one between Alice and Bob.

### A Simple Way to Double-Check
WhatsApp even gives users a simple way to verify this security, which is helpful for those who are particularly cautious. In any contact's info screen, there is an "Encryption" button. Tapping it reveals a unique 60-digit number and a QR code .

If Alice and Bob are in the same room, Bob can scan the QR code on Alice's phone. If the codes match, it confirms that the keys they are using are genuine and that no one has interfered with their connection. It's a simple, visual confirmation that their "digital locks" are working perfectly .

### We can do cool things using e2ee, but what does it not cover? 
For completeness, it's also helpful to understand what E2EE doesn't protect. While the content of the message is safe, WhatsApp can see the metadata information about the communication, like who you talked to and when . Also, by default, cloud backups of your chats on Google Drive or iCloud are not encrypted unless you manually turn on a specific setting for an extra layer of protection .

In short, WhatsApp's success lies in making this incredibly complex cryptography process, involving the advanced Signal Protocol, completely invisible to the user, ensuring billions of conversations remain privte by default. 

## 5. Conclusion
End-to-end encryption, as I have illustred through the Alice and Bob communication, provides a roubust framework for digital privacy by ensuring that only the intended recipients cn access the content of messages. By placing the power of encryption and decryption solely on the endpoints, it neutralizes threats and intermediaries. However, the story doesn't end with a secure transmission. 
The challenge of storing this data : protecting it "at rest" without compromising usability, is an active and critical area of research. Innovations like CKS and deterministic encryption schemes represent the ongoing effort to build systems that are not unbreakable but also functional, ensuring that our most private digital conversations remain safe and accessible. 
Just make sure that your product protect it's users conversations form eavesdropping and unauthorised access. 

## 6. References
I might have missed something in this article, so here are some references that can help and give you more information about the same, visit them:

1. Diffie, W., & Hellman, M. (1976). New Directions in Cryptography. IEEE Transactions on Information Theory, 22(6), 644-654.

2. Singh, S. (1999). The Code Book: The Science of Secrecy from Ancient Egypt to Quantum Cryptography. Doubleday.

3. Open Whisper Systems. (2016). Signal Protocol Technical Documentation. Retrieved from https://signal.org/docs/

4. Cohn-Gordon, K., Cremers, C., Dowling, B., Garratt, L., & Stebila, D. (2017). A Formal Security Analysis of the Signal Messaging Protocol. 2017 IEEE European Symposium on Security and Privacy (EuroS&P), 451-466.

5. WhatsApp. (2023). WhatsApp Encryption Overview: Technical White Paper. Meta Platforms, Inc. Retrieved from https://www.whatsapp.com/security/

6. Facebook Engineering. (2020). Compact Key Storage: Keeping secrets safe and accessible. Retrieved from https://engineering.fb.com/security/compact-key-storage/

7. Boneh, D., Di Crescenzo, G., Ostrovsky, R., & Persiano, G. (2004). Public Key Encryption with Keyword Search. International Conference on the Theory and Applications of Cryptographic Techniques (EUROCRYPT), 506-522.

8. Marlinspike, M. (2016). Forward Secrecy for Asynchronous Messages. Signal Blog. Retrieved from https://signal.org/blog/asynchronous-security/

9. Krawczyk, H., & Eronen, P. (2010). HMAC-based Extract-and-Expand Key Derivation Function (HKDF). IETF RFC 5869.

10. WhatsApp Help Center. (2024). About End-to-End Encryption. Retrieved from https://faq.whatsapp.com/general/security-and-privacy/end-to-end-encryption

11. Rudra, G. (2024, December 31). Low-Level Design for an End-to-End Encrypted Messaging Application with Real-Time Communication. Medium. Retrieved from https://medium.com/@gynanrudr0/low-level-design-for-an-end-to-end-encrypted-messaging-application-with-real-time-communication-6aff2ff34459

12. Shelake, S. (2026, January 18). End-to-End Encryption (E2EE) in Chat Applications: A Complete Guide. Medium. Retrieved from https://medium.com/@siddhantshelake/end-to-end-encryption-e2ee-in-chat-applications-a-complete-guide-12b226cae8f8