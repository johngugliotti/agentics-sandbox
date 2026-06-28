Using your PKI (Public Key Infrastructure) certificate to authenticate with internal web services is a very common pattern, often referred to as **Mutual TLS (mTLS)** or client certificate authentication.
The requests library in Python makes this incredibly straightforward using the cert parameter.
Here is how you can set it up, depending on how your certificate files are formatted.
## Scenario 1: You have separate .crt and .key files
If your certificate and private key are in separate files, you pass them as a tuple ('path/to/cert', 'path/to/key').
```python
import requests

# Path to your certificate and private key
cert_file = "path/to/user_cert.crt"
key_file = "path/to/user_key.key"

# The internal endpoint
url = "https://internal-service.yourcompany.com/api/data"

try:
    # Make the request passing the cert tuple
    response = requests.get(url, cert=(cert_file, key_file))
    
    # Check if the request was successful
    response.raise_for_status()
    print("Success! Response text:")
    print(response.text)

except requests.exceptions.HTTPError as err:
    print(f"HTTP Error occurred: {err}")
except requests.exceptions.RequestException as e:
    print(f"An error occurred: {e}")

```
## Scenario 2: You have a single combined .pem file
If your certificate and private key are combined into a single .pem file, you can just pass the path to that single file as a string.
```python
import requests

# Path to the combined certificate and key file
combined_cert = "path/to/combined_user.pem"
url = "https://internal-service.yourcompany.com/api/data"

response = requests.get(url, cert=combined_cert)
print(response.status_code)

```
## Crucial Troubleshooting Tips for Internal Domains
When dealing with internal company domains, you will almost always run into two common hurdles:
### 1. Handling Internal Certificate Authorities (CAs)
Because it's an internal domain, Python might not recognize your company's root CA, resulting in an SSLCertVerificationError.
 * **The Quick (but unsafe) Fix:** Turn off verification using verify=False. *Do not use this in production*, as it disables SSL verification entirely, leaving you open to man-in-the-middle attacks.
   ```python
   response = requests.get(url, cert=(cert_file, key_file), verify=False)
   
   ```
 * **The Correct Fix:** Pass the path to your company’s root CA bundle using the verify parameter.
   ```python
   company_ca_bundle = "path/to/company_root_ca.pem"
   response = requests.get(url, cert=(cert_file, key_file), verify=company_ca_bundle)
   
   ```
### 2. If your certificate has a password / passphrase
The requests library does not natively support password-protected private keys out of the box via the cert parameter. If your PKI key requires a password, you have two options:
 1. **Remove the passphrase** (if allowed by your security policy) via OpenSSL:
   ```bash
   openssl rsa -in user_key.key -out decrypted_user_key.key
   
   ```
 2. **Use an SSL Context** with urllib3 or a custom transport adapter in requests to load the password-protected key.
Are your certificate files currently password-protected, or are you running into any specific SSL handshake errors?
