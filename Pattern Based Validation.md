Certainly! Here are more **India-specific patterns** you can validate using regular expressions, all of which can be handled inline without helper functions.

I'll add the following new ones:

---

### 🇮🇳 **Additional India-Specific Inputs to Support:**

| Type              | Format / Example      | Notes                                         |
| ----------------- | --------------------- | --------------------------------------------- |
| `passport`        | A1234567              | 1 letter followed by 7 digits                 |
| `driving_license` | DL01 2023001234567    | State code + RTO + year + number (simplified) |
| `upi_id`          | name\@bank            | UPI handles                                   |
| `tan`             | ABCD12345E            | 4 letters, 5 digits, 1 letter                 |
| `esic_number`     | 12345678901234567     | 17 digits                                     |
| `pf_uan`          | 123456789012          | 12-digit UAN                                  |
| `cin_number`      | U12345MH2000PTC123456 | 21-character company CIN                      |

---

### ✅ Updated Code With More India-Specific Patterns:

```python
import re

def validate_input(input_type: str, value: str, custom_pattern: str = None) -> bool:
    """
    Validates a wide range of input types using regex patterns.
    Allows interactive regex input if custom pattern is not provided for 'custom' type.

    Parameters:
        input_type (str): Type of input (e.g., 'email', 'aadhaar', 'custom')
        value (str): Input value to validate
        custom_pattern (str): Regex pattern for 'custom' type

    Returns:
        bool: True if valid, False otherwise
    """
    
    def _ask_user_pattern():
        print("⚠️ No custom regex pattern was provided.")
        return input("🔧 Please enter a custom regex pattern to validate the value: ")

    validators = {
        # Global
        'email': lambda v: re.match(r'^[\w\.-]+@[\w\.-]+\.\w+$', v),
        'phone': lambda v: re.match(r'^\+?\d{7,15}$', v),
        'ip': lambda v: re.match(r'^(\d{1,3}\.){3}\d{1,3}$', v),
        'url': lambda v: re.match(r'^(https?://)?[\w.-]+\.[a-zA-Z]{2,}(/[\w./?%&=-]*)?$', v),
        'username': lambda v: re.match(r'^[a-zA-Z0-9_.-]{3,20}$', v),
        'password': lambda v: re.match(r'^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d@$!%*?&]{8,}$', v),
        'hex_color': lambda v: re.match(r'^#(?:[0-9a-fA-F]{3}){1,2}$', v),
        'mac_address': lambda v: re.match(r'^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$', v),
        'ipv6': lambda v: re.match(r'^([0-9a-fA-F]{0,4}:){2,7}[0-9a-fA-F]{0,4}$', v),
        'isbn10': lambda v: re.match(r'^\d{9}[\dX]$', v),
        'isbn13': lambda v: re.match(r'^\d{13}$', v),
        'cvv': lambda v: re.match(r'^\d{3,4}$', v),
        'zipcode_us': lambda v: re.match(r'^\d{5}(-\d{4})?$', v),
        'ssn_us': lambda v: re.match(r'^\d{3}-\d{2}-\d{4}$', v),
        'routing_number_us': lambda v: re.match(r'^\d{9}$', v),
        'twitter_handle': lambda v: re.match(r'^@?(\w){1,15}$', v),
        'vehicle_plate_us': lambda v: re.match(r'^[A-Z0-9]{1,7}$', v),

        # India-specific
        'aadhaar': lambda v: re.match(r'^\d{4} \d{4} \d{4}$', v),
        'pan': lambda v: re.match(r'^[A-Z]{5}[0-9]{4}[A-Z]$', v),
        'gstin': lambda v: re.match(r'^\d{2}[A-Z]{5}\d{4}[A-Z][A-Z\d]Z[A-Z\d]$', v),
        'indian_phone': lambda v: re.match(r'^(\+91[\-\s]?)?[6-9]\d{9}$', v),
        'ifsc': lambda v: re.match(r'^[A-Z]{4}0[A-Z0-9]{6}$', v),
        'pin_code_india': lambda v: re.match(r'^\d{6}$', v),
        'voter_id': lambda v: re.match(r'^[A-Z]{3}\d{7}$', v),
        'passport': lambda v: re.match(r'^[A-PR-WYa-pr-wy][0-9]{7}$', v),
        'driving_license': lambda v: re.match(r'^[A-Z]{2}\d{2} ?\d{4} ?\d{7}$', v),  # e.g., DL01 2023 0012345
        'upi_id': lambda v: re.match(r'^[\w\.\-]{2,256}@[a-zA-Z]{2,64}$', v),
        'tan': lambda v: re.match(r'^[A-Z]{4}[0-9]{5}[A-Z]$', v),
        'esic_number': lambda v: re.match(r'^\d{17}$', v),
        'pf_uan': lambda v: re.match(r'^\d{12}$', v),
        'cin_number': lambda v: re.match(r'^[A-Z]{1}\d{5}[A-Z]{2}\d{4}[A-Z]{3}\d{6}$', v),

        # Custom
        'custom': lambda v: re.match(custom_pattern or _ask_user_pattern(), v)
    }

    validator = validators.get(input_type.lower())
    return bool(validator(value)) if validator else False
```

---

### ✅ Example Usage of New India-Specific Patterns:

```python
print(validate_input('passport', 'K1234567'))             # ✅ True
print(validate_input('driving_license', 'DL01 2023 1234567'))  # ✅ True
print(validate_input('upi_id', 'rahul123@ybl'))           # ✅ True
print(validate_input('tan', 'ABCD12345E'))                # ✅ True
print(validate_input('pf_uan', '123456789012'))           # ✅ True
print(validate_input('cin_number', 'U12345MH2000PTC123456'))  # ✅ True
```

---

Would you like to generate a quick **CLI tool or a web interface** (like with Flask or Streamlit) for this validation system?
