# FUTURE_CS_01

# Two-Factor Authentication (2FA) Implementation Documentation

## Overview

This documentation details the implementation of Two-Factor Authentication (2FA) in a sample web application using Python and Flask. 2FA enhances account security by requiring users to provide a second form of identification in addition to their password.

## Importance of Two-Factor Authentication

Two-Factor Authentication adds an additional layer of security to user accounts by requiring both a password and a second factor, such as a generated token. This significantly reduces the risk of unauthorized account access through compromised passwords.

### Benefits:

- **Increased Security**: Provides additional protection against unauthorized access.
- **Protection Against Phishing**: Secures user accounts, even if passwords are stolen.
- **Reduction of Account Takeover**: Lowers the chance of account hijacking.
- **User Awareness and Trust**: Increases user confidence in application security.
- **Compliance with Regulations**: Meets security requirements in various industries.

## Prerequisites

- **Python 3.x**: Ensure Python is installed on your system.
- **Flask**: A lightweight WSGI web application framework for Python.
- **Libraries**: We'll be using `pyotp` (for generating TOTP tokens) and `qrcode` (for generating QR codes).

## Environment Setup

1. **Create a New Directory**:
   ```bash
   mkdir two_factor_auth_app
   cd two_factor_auth_app
   ```

2. **Set Up a Virtual Environment** (optional but recommended):
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows use `venv\Scripts\activate`
   ```

3. **Install Required Packages**:
   ```bash
   pip install Flask pyotp qrcode[pil]
   ```

## Implementation Steps

### 1. Create Flask Application

Create a file named `app.py` and set up a basic Flask application to manage user registrations and logins.

```python
from flask import Flask, request, session, render_template_string, redirect, url_for
import pyotp
import qrcode
import io
import base64

app = Flask(__name__)
app.secret_key = 'your_secret_key'  # Change this to a strong secret
user_data = {}  # In-memory user data (for demo purposes)

# Home route
@app.route('/')
def home():
    return render_template_string("""
        <h1>Two-Factor Authentication Demo</h1>
        <form action="/register" method="POST">
            <h2>Register</h2>
            <input type="text" name="username" placeholder="Username" required>
            <button type="submit">Register</button>
        </form>
        <form action="/login" method="POST">
            <h2>Login</h2>
            <input type="text" name="username" placeholder="Username" required>
            <input type="text" name="token" placeholder="2FA Token" required>
            <button type="submit">Login</button>
        </form>
    """)

# Register route
@app.route('/register', methods=['POST'])
def register():
    username = request.form['username']
    totp = pyotp.TOTP(pyotp.random_base32())
    user_data[username] = {'secret': totp.secret, 'verified': False}

    # Generate QR code
    uri = totp.provisioning_uri(name=username, issuer='MyApp')
    img = qrcode.make(uri)
    buffer = io.BytesIO()
    img.save(buffer)
    buffer.seek(0)
    qr_code_data = base64.b64encode(buffer.getvalue()).decode('utf-8')

    return render_template_string("""
        <h2>Scan the QR Code with Google Authenticator</h2>
        <img src="data:image/png;base64,{{ qr_code_data }}" alt="QR Code">
        <form action="/verify" method="POST">
            <input type="hidden" name="username" value="{{ username }}">
            <input type="text" name="token" placeholder="Enter token" required>
            <button type="submit">Verify</button>
        </form>
    """, qr_code_data=qr_code_data, username=username)

# Verify 2FA token
@app.route('/verify', methods=['POST'])
def verify():
    username = request.form['username']
    token = request.form['token']
    user = user_data.get(username)

    if user:
        totp = pyotp.TOTP(user['secret'])
        if totp.verify(token):
            user['verified'] = True
            session['username'] = username
           
