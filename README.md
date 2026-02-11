# **Secure Banking ATM System**

## **Overview**
The Secure Banking ATM System is a client-server application designed to simulate the functionality of an ATM service while emphasizing network security principles. The project implements secure communication protocols to protect sensitive financial transactions, ensuring confidentiality, integrity, and authenticity of data exchanged between the ATM client and the bank server. The system supports essential banking operations such as account registration, deposits, withdrawals, and balance inquiries, while maintaining a robust security layer to prevent unauthorized access and tampering.

This project demonstrates practical implementation of cryptographic protocols, secure key derivation, authentication mechanisms, and secure session management in a networked environment.

---

## **Table of Contents**
- [Features](#features)
- [System Architecture](#system-architecture)
- [Project Structure](#project-structure)
- [Security Features](#security-features)
- [Cryptographic Protocol](#cryptographic-protocol)
- [System Components](#system-components)
- [Setup Instructions](#setup-instructions)
- [Usage Guide](#usage-guide)
- [API Reference](#api-reference)
- [Testing and Demo](#testing-and-demo)
- [Technical Specifications](#technical-specifications)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)
- [Future Enhancements](#future-enhancements)
- [Contributing](#contributing)
- [License](#license)

---

## **Features**
1. **Secure Account Registration**:
   - Users can register with a username and password
   - Passwords are hashed using bcrypt with automatic salt generation
   - Duplicate username prevention
   - Initial account balance set to $0

2. **Secure ATM Login**:
   - Multi-factor authentication using username and password
   - Password verification using bcrypt hashing
   - Failed login attempts are logged
   - Session-based authentication

3. **Secure Authentication and Key Exchange**:
   - Master Secret generation using cryptographically secure random bytes (os.urandom)
   - Key Derivation Function (HKDF) with SHA-256 for session key generation
   - Two session keys are derived from the `Master Secret`:
     - **Encryption Key** (32 bytes): Ensures confidentiality of data using Fernet symmetric encryption
     - **MAC Key** (32 bytes): Ensures integrity and authenticity using HMAC-SHA256

4. **Secure Transaction Processing**:
   - Supports three types of transactions:
     - **Deposits**: Add funds to account balance
     - **Withdrawals**: Withdraw funds with balance validation
     - **Balance Inquiries**: Check current account balance
   - All transactions are encrypted using Fernet encryption (AES-128 in CBC mode)
   - Message Authentication Code (MAC) is generated and verified for every transaction
   - Prevents tampering and replay attacks

5. **Comprehensive Audit Logging**:
   - Dual logging system:
     - Encrypted audit log (`audit_log.enc`) - for secure archival
     - Plaintext audit log (`audit_log_unencrypted.txt`) - for debugging
   - Each log entry includes:
     - Username/Client ID
     - Action performed
     - Timestamp (YYYY-MM-DD HH:MM:SS format)
   - Thread-safe logging with database locks

6. **Multi-Client Concurrency**:
   - Server supports multiple concurrent clients using Python threading
   - Thread-safe database operations with locks
   - Each client maintains a persistent connection during their session
   - Independent client instances with unique port assignments

7. **User-Friendly Web Interface**:
   - Flask-based web application for ATM client
   - Responsive HTML/CSS interface with custom styling
   - Three main pages:
     - Login page
     - Signup page
     - Transaction/Action page
   - Real-time feedback on transaction results
   - Session management with username display

---

## **System Architecture**

The system follows a client-server architecture with the following components:

```
┌─────────────────────────────────────────────────────────────┐
│                     ATM Client (Flask)                       │
│  ┌────────────┐  ┌────────────┐  ┌────────────────────┐    │
│  │  Login UI  │  │ Signup UI  │  │  Transaction UI    │    │
│  └────────────┘  └────────────┘  └────────────────────┘    │
│         │                │                  │               │
│         └────────────────┴──────────────────┘               │
│                          │                                  │
│              ┌───────────▼──────────┐                       │
│              │  Client Socket Layer │                       │
│              │  (Encryption/MAC)    │                       │
│              └───────────┬──────────┘                       │
└──────────────────────────┼──────────────────────────────────┘
                           │
                    TCP/IP Connection
                    (127.0.0.1:65432)
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                    Bank Server (Python)                      │
│              ┌───────────────────────┐                       │
│              │  Server Socket Layer  │                       │
│              │  (Multi-threaded)     │                       │
│              └───────────┬───────────┘                       │
│                          │                                   │
│      ┌──────────────────┼──────────────────┐               │
│      │                  │                  │               │
│ ┌────▼────┐   ┌─────────▼────────┐   ┌────▼─────┐        │
│ │  Auth   │   │  Key Derivation  │   │ Transaction│        │
│ │ Handler │   │  (HKDF-SHA256)   │   │  Handler  │        │
│ └─────────┘   └──────────────────┘   └───────────┘        │
│      │                                      │               │
│      └──────────────┬───────────────────────┘               │
│                     │                                       │
│            ┌────────▼────────┐                              │
│            │  Database Layer │                              │
│            │  (In-Memory)    │                              │
│            │  • users {}     │                              │
│            │  • accounts {}  │                              │
│            └─────────────────┘                              │
│                     │                                       │
│            ┌────────▼────────┐                              │
│            │  Audit Logger   │                              │
│            │  (Dual Format)  │                              │
│            └─────────────────┘                              │
└─────────────────────────────────────────────────────────────┘
```

### **Communication Flow**

1. **Registration Phase**:
   - Client sends username and password
   - Server validates uniqueness and creates account
   - Password is hashed using bcrypt before storage

2. **Authentication Phase**:
   - Client sends login credentials
   - Server validates credentials using bcrypt.checkpw()
   - Upon success, server generates Master Secret
   - Both parties derive encryption and MAC keys using HKDF

3. **Transaction Phase**:
   - Client encrypts transaction payload with encryption key
   - Client generates MAC of encrypted payload
   - Server verifies MAC and decrypts payload
   - Server processes transaction and sends encrypted response
   - All transactions are logged to audit files

---

## **Project Structure**

```
Network-Security-Project/
├── atm_client.py              # Flask-based ATM client application
├── bank_server.py             # Multi-threaded bank server
├── templates/                 # HTML templates for web interface
│   ├── login.html            # Login page
│   ├── signup.html           # Registration page
│   └── action.html           # Transaction page
├── static/                    # CSS and static assets
│   ├── styles.css            # Global styles
│   ├── login.css             # Login page styles
│   ├── signup.css            # Signup page styles
│   ├── action.css            # Transaction page styles
│   └── bground.jpg           # Background image
├── COE817 Project Report.pdf  # Detailed project documentation
├── COE817Project.zip          # Project archive
├── .gitignore                # Git ignore rules
└── README.md                 # This file
```

### **Key Files Description**

- **atm_client.py** (155 lines):
  - Flask web server implementation
  - Handles user interface and client-side logic
  - Manages encryption and MAC generation
  - Maintains persistent socket connection
  - Port management for multiple clients

- **bank_server.py** (180 lines):
  - Multi-threaded socket server
  - User authentication and session management
  - Transaction processing (deposit, withdrawal, balance)
  - Key derivation and secure communication
  - Dual-format audit logging

---

## **Security Features**

### **1. Confidentiality**
   - **Encryption Algorithm**: Fernet (symmetric encryption)
     - Uses AES-128 in CBC mode
     - PKCS7 padding
     - HMAC authentication of ciphertext
   - **Key Size**: 256-bit encryption key (base64 encoded from 32 bytes)
   - **Key Derivation**: HKDF with SHA-256
   - All transaction data is encrypted before transmission
   - Master Secret is transmitted only once per session

### **2. Integrity**
   - **MAC Algorithm**: HMAC-SHA256
   - **MAC Key Size**: 256 bits (32 bytes)
   - Every message includes a MAC for integrity verification
   - Server rejects messages with invalid MACs
   - Prevents message tampering and modification attacks

### **3. Authentication**
   - **Password Hashing**: bcrypt with automatic salt generation
   - **Work Factor**: Default bcrypt rounds (cost factor)
   - Client and server mutually authenticate during key exchange
   - Session-based authentication after initial login
   - Failed authentication attempts are logged

### **4. Replay Protection**
   - Unique Master Secret generated for each session
   - Session keys are derived per login session
   - Prevents replay of old encrypted messages
   - Connection-based session management

### **5. Audit and Accountability**
   - Comprehensive transaction logging
   - Encrypted audit logs prevent tampering
   - Plaintext logs for debugging and analysis
   - Timestamps on all logged events
   - Client identification in logs

---

## **Cryptographic Protocol**

### **Key Derivation Process**

1. **Master Secret Generation**:
   ```python
   master_secret = os.urandom(32)  # 256-bit random secret
   ```

2. **Key Derivation using HKDF**:
   ```python
   hkdf = HKDF(
       algorithm=SHA256(),
       length=64,           # Derive 64 bytes total
       salt=None,           # No salt used
       info=b'ATM Session Keys'  # Context-specific info
   )
   derived_keys = hkdf.derive(master_secret)
   encryption_key = derived_keys[:32]   # First 32 bytes
   mac_key = derived_keys[32:]          # Last 32 bytes
   ```

3. **Encryption Key Encoding**:
   ```python
   encryption_key = base64.urlsafe_b64encode(encryption_key)
   # Required for Fernet specification
   ```

### **Message Format**

**Client → Server (Transaction Request)**:
```json
{
    "payload": "<base64-encoded-encrypted-data>",
    "mac": "<hex-encoded-hmac-sha256>"
}
```

**Encrypted Payload Content**:
```json
{
    "action": "deposit|withdraw|balance",
    "amount": 100  // Optional, not required for balance
}
```

**Server → Client (Transaction Response)**:
```
<fernet-encrypted-response>
```

**Decrypted Response Examples**:
- `"Deposited $100. New Balance: $250"`
- `"Withdrew $50. New Balance: $200"`
- `"Current Balance: $300"`
- `"Insufficient funds"`
- `"MAC verification failed"`

---

## **System Components**

### **1. Bank Server (`bank_server.py`)**
The bank server is a multi-threaded application that listens for incoming client connections, authenticates users, and processes transactions securely.

#### **Key Features**:
- **Multi-Threading**:
  - Each client connection handled in a separate thread
  - Thread-safe database operations using locks
  - Supports unlimited concurrent clients
  
- **Authentication System**:
  - Passwords hashed using bcrypt with automatic salt generation
  - Secure password verification with bcrypt.checkpw()
  - Username uniqueness validation
  - Authentication failure handling

- **Key Derivation**:
  - HKDF (HMAC-based Key Derivation Function) with SHA-256
  - Derives 64 bytes from Master Secret:
    - 32 bytes for encryption key
    - 32 bytes for MAC key
  - Cryptographically secure random Master Secret generation

- **Transaction Processing**:
  - **Deposit**: Adds amount to account balance
  - **Withdraw**: Deducts amount with balance validation
  - **Balance**: Returns current account balance
  - All operations protected by database lock
  - MAC verification before processing

- **Audit Logging System**:
  - Dual-format logging:
    - `audit_log.enc`: Encrypted using Fernet
    - `audit_log_unencrypted.txt`: Plain text for analysis
  - Each entry includes: username, action, timestamp
  - Append-only log files

- **Security Mechanisms**:
  - MAC verification for all transactions
  - Encrypted response transmission
  - Session-based key management
  - Connection state management

#### **Server Configuration**:
- **Host**: `127.0.0.1` (localhost)
- **Port**: `65432`
- **Protocol**: TCP/IP
- **Encoding**: UTF-8
- **Buffer Size**: 4096 bytes

---

### **2. ATM Client (`atm_client.py`)**
The ATM client is a Flask-based web application that provides the user interface for the ATM service.

#### **Key Features**:

- **Web Interface (Flask)**:
  - Three main routes:
    - `/` (GET): Login page
    - `/signup` (GET/POST): Registration page
    - `/login` (POST): Authentication endpoint
    - `/action` (POST): Transaction processing
  - Template rendering with Jinja2
  - Form-based user interaction
  - Session management with global variables

- **Client Instance Management**:
  - Dynamic port assignment: `5000 + client_number`
  - Command-line argument for client number
  - Port availability checking
  - Prevents port conflicts
  - Example: Client 1 runs on port 5001, Client 2 on port 5002

- **Secure Communication**:
  - Fernet encryption for all transaction data
  - HMAC-SHA256 MAC generation
  - Encrypted payload transmission
  - MAC verification of responses
  - Base64 encoding for transport

- **Session Management**:
  - Persistent socket connection during session
  - Global session keys (encryption_key, mac_key)
  - Username tracking (current_username)
  - Socket lifecycle management
  - Connection cleanup on errors

- **Transaction Handling**:
  - Form-based amount input
  - Action type selection (deposit/withdraw/balance)
  - Payload encryption before transmission
  - Response decryption and display
  - Error handling and user feedback

#### **Client Configuration**:
- **Server Host**: `127.0.0.1`
- **Server Port**: `65432`
- **Client Port**: `5000 + client_number`
- **Protocol**: TCP/IP
- **Encoding**: UTF-8
- **Buffer Size**: 4096 bytes

---

## **Security Features**
1. **Confidentiality**:
   - All transaction data is encrypted using the encryption key derived from the `Master Secret`.

2. **Integrity**:
   - MACs are used to ensure that transaction data has not been tampered with.

3. **Authentication**:
   - The client and server authenticate each other during the key distribution phase.

4. **Replay Protection**:
   - The use of a unique `Master Secret` for each session prevents replay attacks.

5. **Audit Logging**:
   - Transactions are logged in both plaintext and encrypted formats for accountability and debugging.

---

## **Setup Instructions**

### **1. Prerequisites**

#### **System Requirements**:
- **Operating System**: Linux, macOS, or Windows
- **Python Version**: Python 3.8 or higher
- **Network**: TCP/IP networking support
- **Storage**: Minimal (< 10 MB for application)

#### **Required Python Libraries**:
```bash
pip install bcrypt cryptography Flask
```

**Library Versions** (tested with):
- `bcrypt>=4.0.0` - Password hashing
- `cryptography>=41.0.0` - Fernet encryption and HKDF
- `Flask>=2.3.0` - Web framework for client UI

**Dependency Details**:
- **bcrypt**: Secure password hashing with adaptive cost
- **cryptography**: Provides Fernet symmetric encryption and HKDF key derivation
- **Flask**: Lightweight web framework for the ATM client interface

### **2. Installation**

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Jahswaygo/Network-Security-Project.git
   cd Network-Security-Project
   ```

2. **Create Virtual Environment** (Recommended):
   ```bash
   python -m venv venv
   
   # On Linux/macOS:
   source venv/bin/activate
   
   # On Windows:
   venv\Scripts\activate
   ```

3. **Install Dependencies**:
   ```bash
   pip install bcrypt cryptography Flask
   ```

4. **Verify Installation**:
   ```bash
   python -c "import bcrypt, cryptography, flask; print('All dependencies installed successfully')"
   ```

---

### **3. Running the Application**

#### **Step 1: Start the Bank Server**

1. Open a terminal window
2. Navigate to the project directory
3. Run the server:
   ```bash
   python bank_server.py
   ```

**Expected Output**:
```
[LISTENING] on 127.0.0.1:65432
```

The server will:
- Listen on `127.0.0.1:65432`
- Create audit log files on first transaction
- Accept multiple concurrent client connections
- Run until manually terminated (Ctrl+C)

**Server Logs**:
- `[CONNECTED] <address>` - New client connection
- `[DISCONNECTED] <address>` - Client disconnection
- `<address>: Signup successful for username '<user>'`
- `<address>: Login successful for username '<user>'`
- `<address>: Deposit/Withdrawal/Balance inquiry successful`

---

#### **Step 2: Start the ATM Client(s)**

1. Open a **new** terminal window (keep server running)
2. Navigate to the project directory
3. Run the client with a unique client number:
   ```bash
   python atm_client.py <client_number>
   ```
   - Replace `<client_number>` with a unique integer (e.g., `1`, `2`, `3`)
   - Each client needs a unique number to avoid port conflicts

**Examples**:
```bash
# Terminal 2 - Client 1
python atm_client.py 1
# Runs on http://127.0.0.1:5001

# Terminal 3 - Client 2 (optional)
python atm_client.py 2
# Runs on http://127.0.0.1:5002

# Terminal 4 - Client 3 (optional)
python atm_client.py 3
# Runs on http://127.0.0.1:5003
```

**Expected Output**:
```
Starting ATM Client 1 on port 5001...
 * Serving Flask app 'atm_client'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://127.0.0.1:5001
```

---

#### **Step 3: Access the Web Interface**

1. Open a web browser
2. Navigate to the client URL:
   - Client 1: `http://127.0.0.1:5001`
   - Client 2: `http://127.0.0.1:5002`
   - Client 3: `http://127.0.0.1:5003`

3. You should see the ATM Login page

**Note**: The port number is calculated as `5000 + client_number`

---

## **Usage Guide**

### **1. Account Registration (Signup)**

1. Navigate to the signup page:
   - Click "Sign Up" link on the login page, or
   - Go directly to `http://127.0.0.1:5001/signup`

2. Enter your credentials:
   - **Username**: Choose a unique username (alphanumeric recommended)
   - **Password**: Choose a strong password

3. Click "Sign Up" button

4. **Success**: You'll be redirected to the login page with a success message
   - Initial account balance: $0

5. **Error Cases**:
   - "Username already exists" - Choose a different username
   - Connection errors - Ensure the server is running

---

### **2. Login**

1. Navigate to the login page:
   - `http://127.0.0.1:5001/` (or your client's port)

2. Enter your credentials:
   - **Username**: Your registered username
   - **Password**: Your password

3. Click "Login" button

4. **Success**: Redirected to transaction page
   - A persistent connection is established
   - Session keys are derived
   - Your username is displayed on the page

5. **Error Cases**:
   - "Authentication Failed" - Incorrect username or password
   - Connection errors - Ensure the server is running

---

### **3. Performing Transactions**

After logging in, you'll see the transaction page with three options:

#### **A. Deposit Money**
1. Select "Deposit" from the action dropdown
2. Enter the amount to deposit (positive integer)
3. Click "Submit"
4. **Result**: `"Deposited $<amount>. New Balance: $<balance>"`

**Example**:
- Action: Deposit
- Amount: 500
- Result: "Deposited $500. New Balance: $500"

#### **B. Withdraw Money**
1. Select "Withdraw" from the action dropdown
2. Enter the amount to withdraw (positive integer)
3. Click "Submit"
4. **Success**: `"Withdrew $<amount>. New Balance: $<balance>"`
5. **Insufficient Funds**: `"Insufficient funds"`

**Example**:
- Current Balance: $500
- Action: Withdraw
- Amount: 200
- Result: "Withdrew $200. New Balance: $300"

#### **C. Check Balance**
1. Select "Balance" from the action dropdown
2. No amount needed (leave blank or enter any value)
3. Click "Submit"
4. **Result**: `"Current Balance: $<balance>"`

**Example**:
- Action: Balance
- Result: "Current Balance: $300"

---

### **4. Multiple Concurrent Sessions**

You can run multiple clients simultaneously to simulate multiple ATM machines:

**Scenario**: Two users accessing their accounts simultaneously

```bash
# Terminal 1: Server
python bank_server.py

# Terminal 2: Alice's ATM (Client 1)
python atm_client.py 1
# Access: http://127.0.0.1:5001

# Terminal 3: Bob's ATM (Client 2)
python atm_client.py 2
# Access: http://127.0.0.1:5002
```

Each client can:
- Register different users
- Login independently
- Perform transactions concurrently
- Have separate encrypted sessions

---

### **5. Session Management**

- **Active Session**: Maintains connection until browser is closed or page is refreshed
- **Session Keys**: Valid only for the current session
- **Re-login**: Required after closing the browser or reloading the page
- **Security**: Each login generates new session keys

---

## **API Reference**

### **Server Endpoints (Internal)**

The server uses socket-based communication, not HTTP REST API. The following actions are supported:

#### **1. Signup Action**
**Request Format**:
```json
{
    "action": "signup",
    "username": "string",
    "password": "string"
}
```
**Response**: String
- `"Signup Successful"` - Account created
- `"Username already exists"` - Username taken

---

#### **2. Login Action**
**Request Format**:
```json
{
    "action": "login",
    "username": "string",
    "password": "string"
}
```
**Response Sequence**:
1. `"Authenticated"` or `"Authentication Failed"` (string)
2. If authenticated: `<master_secret>` (32 bytes, binary)

---

#### **3. Transaction Actions**
**Request Format** (after encryption and MAC):
```json
{
    "payload": "<encrypted_json>",
    "mac": "<hmac_hex>"
}
```

**Encrypted Payload Structure**:
```json
{
    "action": "deposit" | "withdraw" | "balance",
    "amount": integer  // Optional for balance action
}
```

**Response**: Encrypted string containing result message

**Possible Responses** (decrypted):
- `"Deposited $<amount>. New Balance: $<balance>"`
- `"Withdrew $<amount>. New Balance: $<balance>"`
- `"Current Balance: $<balance>"`
- `"Insufficient funds"`
- `"MAC verification failed"`

---

### **Client Routes (HTTP)**

#### **GET /**
- **Description**: Display login page
- **Returns**: HTML login form

#### **GET /signup**
- **Description**: Display signup page
- **Returns**: HTML signup form

#### **POST /signup**
- **Parameters**:
  - `username` (form data)
  - `password` (form data)
- **Returns**: HTML with result message

#### **POST /login**
- **Parameters**:
  - `username` (form data)
  - `password` (form data)
- **Returns**: Transaction page or error message

#### **POST /action**
- **Parameters**:
  - `action` (form data): "deposit" | "withdraw" | "balance"
  - `amount` (form data, optional): integer
- **Returns**: HTML with transaction result

---

## **Testing and Demo**

### **Demo Scenario: Complete Transaction Flow**

Follow this step-by-step demo to test all features:

#### **Setup (1 minute)**:
```bash
# Terminal 1: Start server
python bank_server.py

# Terminal 2: Start client
python atm_client.py 1
```

#### **Test Case 1: User Registration (1 minute)**:
1. Open browser: `http://127.0.0.1:5001`
2. Click "Sign Up"
3. Enter:
   - Username: `alice`
   - Password: `SecurePass123`
4. Click "Sign Up"
5. **Expected**: Redirected to login page with success message

#### **Test Case 2: User Login (30 seconds)**:
1. Enter:
   - Username: `alice`
   - Password: `SecurePass123`
2. Click "Login"
3. **Expected**: Transaction page displays with username "alice"

#### **Test Case 3: Balance Inquiry (30 seconds)**:
1. Select "Balance" from dropdown
2. Click "Submit"
3. **Expected**: "Current Balance: $0"

#### **Test Case 4: Deposit (30 seconds)**:
1. Select "Deposit"
2. Enter amount: `1000`
3. Click "Submit"
4. **Expected**: "Deposited $1000. New Balance: $1000"

#### **Test Case 5: Withdrawal (30 seconds)**:
1. Select "Withdraw"
2. Enter amount: `300`
3. Click "Submit"
4. **Expected**: "Withdrew $300. New Balance: $700"

#### **Test Case 6: Insufficient Funds (30 seconds)**:
1. Select "Withdraw"
2. Enter amount: `10000`
3. Click "Submit"
4. **Expected**: "Insufficient funds"

#### **Test Case 7: Multiple Clients (2 minutes)**:
```bash
# Terminal 3: Start second client
python atm_client.py 2
```
1. Open browser: `http://127.0.0.1:5002`
2. Register new user: `bob`
3. Login as `bob`
4. Perform transactions on both clients simultaneously
5. **Expected**: Both clients work independently

#### **Test Case 8: Audit Logs (30 seconds)**:
1. Stop the server (Ctrl+C)
2. Check files created:
   ```bash
   cat audit_log_unencrypted.txt
   ```
3. **Expected**: Log entries with timestamps and actions

---

### **Security Testing**

#### **Test 1: Password Hashing**:
```bash
# Try to view stored password (should be hashed)
# In Python interactive shell with server running:
python
>>> import bank_server
>>> print(bank_server.users)  # Will show hashed passwords
```

#### **Test 2: Encrypted Communication**:
Monitor network traffic to verify encryption (requires tools like Wireshark):
- All transaction data should be encrypted
- Passwords not visible in plaintext

#### **Test 3: MAC Verification**:
Attempt to modify encrypted payload manually:
- Server should reject with "MAC verification failed"

---

## **Technical Specifications**

### **Cryptographic Algorithms**

| Component | Algorithm | Key Size | Notes |
|-----------|-----------|----------|-------|
| Password Hashing | bcrypt | - | Adaptive cost, auto-salt |
| Key Derivation | HKDF-SHA256 | 64 bytes output | 32 bytes each for enc/MAC |
| Symmetric Encryption | Fernet (AES-128-CBC) | 256-bit | Includes HMAC |
| MAC | HMAC-SHA256 | 256-bit key | Applied to encrypted data |
| Random Generation | os.urandom | 256-bit | CSPRNG for Master Secret |

### **Network Specifications**

| Parameter | Value |
|-----------|-------|
| Protocol | TCP/IP |
| Server Address | 127.0.0.1 (localhost) |
| Server Port | 65432 |
| Client Ports | 5000 + client_number |
| Encoding | UTF-8 |
| Buffer Size | 4096 bytes |
| Connection Type | Persistent (per session) |

### **Data Structures**

**Server-Side Storage**:
```python
users = {
    "username": b"<bcrypt_hashed_password>",
    ...
}

accounts = {
    "username": integer_balance,
    ...
}
```

**Audit Log Format** (plaintext):
```
<username>    <action>    <YYYY-MM-DD HH:MM:SS>
```

### **Threading Model**

- **Server**: One thread per client connection
- **Database Access**: Protected by `threading.Lock()`
- **Concurrent Operations**: Thread-safe
- **Client**: Single-threaded Flask application

### **Dependencies Version Matrix**

| Dependency | Minimum Version | Tested Version | Purpose |
|------------|----------------|----------------|---------|
| Python | 3.8 | 3.10+ | Runtime |
| bcrypt | 4.0.0 | 4.1.2 | Password hashing |
| cryptography | 41.0.0 | 42.0.0 | Encryption/HKDF |
| Flask | 2.3.0 | 3.0.0 | Web framework |

---

## **Security Considerations**

### **Strengths**

1. **Strong Password Protection**:
   - bcrypt hashing with automatic salt
   - Adaptive cost factor prevents brute-force attacks
   - Secure password storage

2. **Secure Key Exchange**:
   - Master Secret generated with cryptographically secure random
   - HKDF ensures proper key derivation
   - Separate keys for encryption and authentication

3. **Data Confidentiality**:
   - All transaction data encrypted with Fernet
   - AES-128 in CBC mode with HMAC
   - Keys never transmitted after initial exchange

4. **Data Integrity**:
   - HMAC-SHA256 for message authentication
   - MAC verification prevents tampering
   - Encrypted audit logs

5. **Session Security**:
   - Unique session keys per login
   - Persistent connection during session
   - Session isolation between clients

### **Known Limitations**

1. **Master Secret Transmission**:
   - Master Secret transmitted in plaintext over TCP
   - **Mitigation**: Use TLS/SSL in production
   - **Risk**: Man-in-the-middle attacks on local network

2. **No Certificate-Based Authentication**:
   - No PKI infrastructure
   - Relies on pre-shared trust
   - **Mitigation**: Implement mutual TLS authentication

3. **In-Memory Database**:
   - Data lost on server restart
   - No persistence between sessions
   - **Mitigation**: Implement database storage (SQLite, PostgreSQL)

4. **No Rate Limiting**:
   - Vulnerable to brute-force login attempts
   - No account lockout mechanism
   - **Mitigation**: Implement rate limiting and account lockout

5. **Localhost Only**:
   - Configured for 127.0.0.1 only
   - Not designed for production deployment
   - **Mitigation**: Use proper network security and TLS for production

6. **Debug Mode**:
   - Flask runs in development mode
   - Verbose error messages
   - **Mitigation**: Disable debug mode in production

7. **No Transaction Rollback**:
   - No transaction atomicity guarantees
   - Potential for inconsistent state on errors
   - **Mitigation**: Implement proper transaction handling

### **Best Practices Implemented**

✅ Password hashing with bcrypt  
✅ Separate encryption and authentication keys  
✅ MAC verification before processing  
✅ Audit logging for accountability  
✅ Thread-safe database access  
✅ Input validation for amounts  
✅ Error handling and logging  
✅ Session-based key management  

### **Security Recommendations for Production**

1. **Transport Security**:
   - Implement TLS/SSL for all communications
   - Use proper certificate validation
   - Consider mutual TLS authentication

2. **Key Management**:
   - Implement proper key rotation
   - Use hardware security modules (HSM) for key storage
   - Implement key escrow for audit access

3. **Database Security**:
   - Use encrypted database storage
   - Implement proper backup and recovery
   - Use prepared statements to prevent injection

4. **Access Control**:
   - Implement role-based access control (RBAC)
   - Add multi-factor authentication (MFA)
   - Implement session timeouts

5. **Monitoring and Alerting**:
   - Real-time fraud detection
   - Anomaly detection for transactions
   - Security event monitoring (SIEM)

6. **Compliance**:
   - Follow PCI DSS standards for payment systems
   - Implement proper data retention policies
   - Regular security audits and penetration testing

---

## **Troubleshooting**

### **Common Issues and Solutions**

#### **Issue 1: Port Already in Use**
**Error**: `Error: Port 5001 is already in use. Please choose a different client number.`

**Solutions**:
1. Use a different client number:
   ```bash
   python atm_client.py 2  # Try client 2 instead
   ```
2. Terminate the process using the port (if needed)

---

#### **Issue 2: Server Not Running**
**Error**: `ConnectionRefusedError: [Errno 111] Connection refused`

**Solution**:
1. Ensure the server is running:
   ```bash
   python bank_server.py
   ```
2. Check if server is listening:
   ```bash
   netstat -an | grep 65432
   ```

---

#### **Issue 3: Import Errors**
**Error**: `ModuleNotFoundError: No module named 'bcrypt'`

**Solution**:
```bash
pip install bcrypt cryptography Flask
```

---

#### **Issue 4: Session Key Errors**
**Error**: `Error: Please log in first.`

**Cause**: Session keys not initialized or page was refreshed

**Solution**:
1. Return to login page
2. Log in again to establish new session
3. Don't refresh the transaction page during a session

---

#### **Issue 5: MAC Verification Failed**
**Error**: `MAC verification failed`

**Solution**:
1. Log out and log back in
2. Check network connection
3. Restart both client and server if persistent

---

### **Debugging Tips**

1. **Enable Verbose Logging**: Server prints debug information
2. **Check Audit Logs**: `tail -f audit_log_unencrypted.txt`
3. **Monitor Connections**: `netstat -an | grep 65432`

---

## **Future Enhancements**

### **Security Enhancements**
1. ✨ **TLS/SSL Implementation**: Encrypt Master Secret transmission
2. ✨ **Multi-Factor Authentication**: Add MFA support
3. ✨ **Key Rotation**: Implement key rotation mechanism
4. ✨ **Rate Limiting**: Add login attempt limiting

### **Functional Enhancements**
1. 💡 **Database Persistence**: SQLite or PostgreSQL integration
2. 💡 **Transaction History**: View past transactions
3. 💡 **Account Transfers**: Transfer between accounts
4. 💡 **Admin Interface**: User management dashboard

### **Performance Enhancements**
1. ⚡ **Scalability**: Connection pooling and load balancing
2. ⚡ **Optimization**: Asynchronous I/O

---

## **Contributing**

We welcome contributions to improve the Secure Banking ATM System!

### **How to Contribute**

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a Pull Request

### **Contribution Guidelines**

- ✅ Follow PEP 8 style guide for Python code
- ✅ Write clear commit messages
- ✅ Test all changes before submitting
- ✅ Update documentation for new features
- ✅ Ensure security best practices
- ✅ No hardcoded credentials or secrets

---

## **License**

This project is licensed under the MIT License - see the LICENSE file for details.

### **Academic Use**

This project was developed as part of COE817 (Network Security) coursework for educational purposes to demonstrate:
- Secure communication protocols
- Cryptographic implementations
- Client-server architecture
- Network security principles

### **Disclaimer**

⚠️ **This is a demonstration project for educational purposes only.**

**DO NOT use this system in production or with real financial data.**

The system has known limitations and is not designed for:
- Production banking systems
- Real financial transactions
- Handling sensitive customer data
- Compliance with financial regulations (PCI DSS, etc.)

---

## **Acknowledgments**

- **Course**: COE817 - Network Security
- **Libraries Used**:
  - [bcrypt](https://github.com/pyca/bcrypt/) - Password hashing
  - [cryptography](https://github.com/pyca/cryptography) - Encryption and key derivation
  - [Flask](https://flask.palletsprojects.com/) - Web framework

### **References**

1. **Cryptographic Standards**:
   - NIST SP 800-108: Recommendation for Key Derivation Functions
   - RFC 5869: HKDF
   - RFC 2104: HMAC

2. **Security Best Practices**:
   - OWASP Top 10
   - PCI DSS Standards
   - NIST Cybersecurity Framework

---

## **Project Status**

**Current Version**: 1.0.0 (Educational Demo)  
**Status**: ✅ Completed for educational purposes  
**Last Updated**: 2026-02-11

---

## **Contact**

For questions, suggestions, or issues:
- **GitHub Issues**: [Report an issue](https://github.com/Jahswaygo/Network-Security-Project/issues)
- **Repository**: [Network-Security-Project](https://github.com/Jahswaygo/Network-Security-Project)

---

## **Conclusion**

The Secure Banking ATM System successfully demonstrates the implementation of secure communication protocols in a client-server architecture. By integrating advanced cryptographic techniques and adhering to network security best practices, the system ensures the confidentiality, integrity, and authenticity of financial transactions.

### **Key Achievements**

✅ **Secure Communication**: End-to-end encryption of all transactions  
✅ **Strong Authentication**: bcrypt password hashing and session management  
✅ **Data Integrity**: HMAC-SHA256 verification of all messages  
✅ **Audit Trail**: Comprehensive logging for accountability  
✅ **Concurrent Access**: Multi-threaded server supporting multiple clients  
✅ **User-Friendly Interface**: Intuitive web-based GUI  

This project serves as a practical application of network security principles, providing a solid foundation for understanding:
- Cryptographic protocol implementation
- Secure key derivation and management
- Authentication and authorization mechanisms
- Secure client-server communication
- Audit logging and accountability

The system demonstrates that security and usability can coexist, offering valuable insights for further exploration and innovation in cybersecurity.

---

**⭐ If you found this project helpful, please consider giving it a star on GitHub!**
