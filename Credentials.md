Excellent! Let's now enhance the function so it can:

### ✅ Return **only specific credential fields** (e.g. `apikey`, `username`, etc.)

### ✅ Prompt for **only missing fields** if needed

### ✅ Work with a **dictionary of multiple credential fields**

---

## ✅ Final Version: Selective Field Access & Credential Prompting

```python
import os
import json
import uuid
import platform
import hashlib
import base64
from cryptography.fernet import Fernet
import getpass
from pathlib import Path

def manage_credentials(required_fields=None, credential_file: str = None) -> dict:
    """
    Loads encrypted credentials from disk. Prompts for missing fields.
    Returns only the requested fields if specified.
    
    :param required_fields: List of keys to retrieve (e.g., ["apikey"]). If None, returns all.
    :param credential_file: Custom file path to use. Defaults to ~/.config/secure_credentials/credentials.enc
    :return: Dictionary of requested credentials.
    """
    # Use default file if not provided
    if credential_file:
        CREDENTIALS_FILE = Path(credential_file).expanduser().resolve()
    else:
        CREDENTIALS_FILE = Path.home() / ".config" / "secure_credentials" / "credentials.enc"

    # Ensure directory exists
    CREDENTIALS_FILE.parent.mkdir(parents=True, exist_ok=True)

    # Generate machine-specific key
    def generate_machine_key() -> bytes:
        components = [
            platform.node(),
            platform.system(),
            platform.machine(),
            str(uuid.getnode()),
        ]
        fingerprint = "|".join(components)
        sha = hashlib.sha256(fingerprint.encode()).digest()
        return base64.urlsafe_b64encode(sha[:32])

    # Encrypt and store credentials
    def store_credentials(credentials: dict):
        key = generate_machine_key()
        f = Fernet(key)
        encrypted = f.encrypt(json.dumps(credentials).encode())
        with open(CREDENTIALS_FILE, 'wb') as f_out:
            f_out.write(encrypted)

    # Load credentials from encrypted file
    def load_credentials() -> dict | None:
        if not CREDENTIALS_FILE.exists():
            return None
        try:
            key = generate_machine_key()
            f = Fernet(key)
            with open(CREDENTIALS_FILE, 'rb') as f_in:
                encrypted = f_in.read()
            decrypted = f.decrypt(encrypted)
            return json.loads(decrypted.decode())
        except Exception as e:
            print(f"[Error] Failed to load credentials: {e}")
            return None

    # Prompt user for only the required/missing fields
    def prompt_for_missing_fields(existing: dict, required: list) -> dict:
        updated = existing.copy()
        for field in required:
            if field not in updated or not updated[field]:
                if 'password' in field.lower():
                    updated[field] = getpass.getpass(f"Enter {field}: ")
                else:
                    updated[field] = input(f"Enter {field}: ")
        return updated

    # === Main Logic ===
    existing_credentials = load_credentials() or {}
    if required_fields:
        # Only prompt for what's missing
        existing_credentials = prompt_for_missing_fields(existing_credentials, required_fields)
    elif not existing_credentials:
        # No required_fields passed, no existing credentials — prompt for username/password by default
        existing_credentials = prompt_for_missing_fields({}, ["username", "password"])

    # Save updated if new fields were added
    store_credentials(existing_credentials)

    # Return only requested fields or full set
    if required_fields:
        return {key: existing_credentials.get(key) for key in required_fields}
    return existing_credentials
```

---

## ✅ Example Usages:

### 🔑 Get all stored credentials:

```python
creds = manage_credentials()
print(creds)
```

### 🔐 Get only `apikey`:

```python
creds = manage_credentials(required_fields=["apikey"])
print(creds["apikey"])
```

### 🧪 Multiple fields:

```python
creds = manage_credentials(required_fields=["username", "db_password"])
print(creds)
```

If any field is missing, the function will prompt only for that field and then update the encrypted file.

---

## 🛡️ Security & Usability Features:

* Encrypted with a machine-unique key (not stored)
* No unencrypted credentials saved anywhere
* Safe prompting: hides passwords with `getpass`
* Reusable: You can call `manage_credentials()` wherever needed

---

Would you like this function turned into a reusable **Python module** or **CLI command**?
