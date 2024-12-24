
# **DID:451 Method Specification**

## **1. Overview**

The `did:451` method provides a decentralized identifier (DID) format for securely identifying and managing multiple entity types, including:
- **Personas**: Individual identities or roles.
- **Organizations**: Verified or non-verified entities.
- **Documents**: Files such as JSON, PDF, or binary data.
- **Contracts**: Legal or smart contracts, similar to documents, but contracts may have two, or more, cryptographic signatives.
- **Datasets**: Data collections.
- **Objects**: Tangible or intangible assets.
- **Images**: Image files.
- **PDFs**: PDF documents.
- **Binary**: Generic binary files.

This method ensures flexibility for independent personas that have created Publishing Houses not verified against the Domain Name Service on the Internet, and **DNS-verified organizations**, **non-verified entities**, while adhering to strong cryptographic principles.

---

## **2. DID Format**

The `did:451` method conforms to the W3C DID Core Specification. The general format is:

```plaintext
did:451:<subdomain>:<unique-identifier>
```

Where:
- **`did`**: The scheme prefix.
- **`451`**: The registered DID method name.
- **`<subdomain>`**: Represents the entity type. Supported subdomains include:
    - `persona`: Individual personas.
    - `verified persona`: A persona that has been cryptographically verified against an organization (see just below).
    - `organization`: An entity that has been verified against a DNS domain registration through the use of an A record, or a TXT record.
    - `document`: General documents.
    - `contract`: Legal or smart contracts.
    - `dataset`: Data collections.
    - `object`: Tangible or intangible objects.
    - `image`: Image files.
    - `pdf`: PDF documents.
    - `binary`: Binary files.

- **`<unique-identifier>`**: Ensures uniqueness for each DID:
    - For **`persona`**: Human-readable email-like format (`<name>@<label>`).  Personas that are not verified **cannot** use dot notation, and cannot reference a top level domain from the internet.  
    - For **`organization`**: 
        - **DNS-verified**: Use a fully qualified domain (e.g., `example.com`).
        - **Non-verified**: Use a unique lowercase string.
    - For **`document`, `contract`, `dataset`, and `object`**: Use a **GUID**.
    - For **`image`, `pdf`, and `binary`**: Use sanitized filenames (lowercase, no spaces).

---

### **Examples of DIDs**

#### **Personas**
```plaintext
did:451:persona:alice@alicehatesrabbits
did:451:persona:alice@johnsonpharmaceuticals.com
```

#### **Organizations**
- **Non-verified**:
    ```plaintext
    did:451:organization:alicehatesrabbits
    ```
- **DNS-verified**:
    ```plaintext
    did:451:organization:johnsonpharmaceuticals.com
    ```

#### **Documents, Contracts, Datasets, Objects**
```plaintext
did:451:document:123e4567-e89b-12d3-a456-426614174000
did:451:contract:5d4f8e21-3a01-4bc7-a123-1f3b0d93f1f1
did:451:dataset:789abcd-ef01-4567-890a-abcdef123456
did:451:object:car-2345-789abcd
```

#### **Images, PDFs, Binary Files**
```plaintext
did:451:image:img-98765.png
did:451:pdf:file-003.pdf
did:451:binary:blob-789xyz.dat
```

---

## **7. Storage, Proof of Existence, and Validation**

### **7.1 The Four Types of S3 Storage used for Proof of Existence**
1. **Primary Document Stores:**
   - The **Document, Contract, or Transaction Document**
   - The **Metadata Document**
   - The **Green Blockchain DID Document**
   - The **Green Blockchain Block**
    
### **7.2 Use of S3-Compatible Storage for Documents, Personas and Transactions**
1. **Primary Document Storage:**
   - The **DID Document** (e.g., for a persona, published document, or a transaction) is stored in **S3-compatible storage** (e.g., AWS S3, Backblaze B2, Wasabi).
   - Storage URLs are included in the `serviceEndpoint` field of the DID Document. Example:
     ```json
     "service": [
       {
         "id": "did:451:document:123e4567-e89b-12d3-a456-426614174000#storage",
         "type": "DecentralizedStorage",
         "serviceEndpoint": "https://s3.example.com/document-123e4567-e89b-12d3-a456-426614174000"
       }
     ]
     ```

2. **Benefits of S3-Compatible Storage:**
   - **High availability**: Ensures that the document is accessible globally.
   - **Immutable backups**: By preserving the eTag and storage URL, the document's integrity is ensured.

---

### **7.3 Proof of Existence on the Green Blockchain**
1. **Purpose of Blockchain Anchoring:**
   - The **green blockchain** provides proof of existence and timestamping for the DID Document.
   - The document hash, along with its DID and metadata, is recorded in a blockchain block.

2. **Key Blockchain Fields:**
   - **Hash:** A SHA-256 hash of the document ensures immutability and integrity.
   - **DID:** Links the blockchain entry to the specific document or persona.
   - **Timestamp:** Indicates when the proof of existence was recorded.

3. **Verification Process:**
   - When resolving a DID, the resolver retrieves the blockchain entry to:
     - Verify the recorded hash matches the hash of the stored document.
     - Confirm the document’s existence and immutability.

---

### **7.4 Role of ETags in Validation**
1. **Definition of ETag:**
   - An **ETag** is a unique identifier generated by S3-compatible storage services for each stored object.
   - Example: `"etag": "123456789abcdef123456789abcdef12"`

2. **Purpose in DID Document Validation:**
   - The ETag ensures that the document referenced in the DID Document has not been altered since it was stored.
   - The ETag can be included in the blockchain entry as an additional validation mechanism.

3. **Usage in the DID Document:**
   - The ETag can be part of the service metadata in the DID Document:
     ```json
     "service": [
       {
         "id": "did:451:document:123e4567-e89b-12d3-a456-426614174000#storage",
         "type": "DecentralizedStorage",
         "serviceEndpoint": "https://s3.example.com/document-123e4567-e89b-12d3-a456-426614174000",
         "etag": "123456789abcdef123456789abcdef12"
       }
     ]
     ```

---

## **8. Security and Privacy Considerations**
- Ensure encryption and integrity of data.
- Use secure storage for private keys.
