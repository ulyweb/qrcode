>[!NOTE]
> ### **A full HTML+CSS+JavaScript web app**:
>>* Generates QR codes from secret messages, protected by password.
>>* Allows decrypting the QR by entering the password (all in-browser, no server needed).



---

## 🚦 **Important Notes**

* **Crypto:** We'll use the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) for AES-256 encryption/decryption with PBKDF2.
* **QR Code:** We'll use [QRCode.js](https://davidshimjs.github.io/qrcodejs/) to generate QR codes in-browser.
* **Reading QR from image:** We'll use [jsQR](https://github.com/cozmo/jsQR) to decode QR codes from an uploaded PNG/JPG.
* **No data leaves the browser.**

---

# 🌐 **`index.html` - Complete Web App**

Save this file as `index.html` in your repo. When pushed to GitHub Pages, it will work out of the box!

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Password-Protected QR Code Generator & Decoder</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <meta name="color-scheme" content="light dark">
  <style>
    :root {
      --primary: #1967d2; --success: #43a047; --error: #d32f2f;
      --bg: #f4f7fa; --panel: #fff; --text: #222; --border: #d3d3d3;
    }
    html.dark {
      --primary: #8ab4f8; --success: #81c784; --error: #ef5350;
      --bg: #181c23; --panel: #23272f; --text: #eee; --border: #373e4b;
    }
    body { background: var(--bg); color: var(--text); font-family: 'Segoe UI', Arial, sans-serif; margin:0; padding:0; transition: background .3s, color .3s;}
    .container { max-width: 540px; margin: 44px auto 30px auto; background: var(--panel); border-radius: 14px; box-shadow: 0 2px 24px #0002; padding: 32px 30px; transition: background .3s;}
    h1 { text-align: center; color: var(--primary);}
    .tabs { display: flex; margin-bottom: 22px; }
    .tab { flex: 1; text-align: center; padding: 13px; background: var(--bg); cursor: pointer; border-radius: 9px 9px 0 0; border-bottom: 2px solid var(--border); font-weight: 500; letter-spacing: 0.5px; user-select: none; color: var(--primary); transition: background .2s, color .2s;}
    .tab.active { background: var(--panel); border-bottom: 2px solid var(--panel); color: var(--primary); font-weight: bold;}
    .form-group { margin: 19px 0; }
    label { display: block; margin-bottom: 8px; }
    input[type=text], input[type=password], input[type=file], textarea { width: 100%; padding: 11px; border-radius: 7px; border: 1px solid var(--border); font-size: 1rem; background: var(--bg); color: var(--text); transition: background .2s;}
    input[type=file] { padding: 8px;}
    button { background: var(--primary); color: #fff; border: none; padding: 13px 20px; border-radius: 8px; cursor: pointer; font-size: 1rem; margin-top: 8px; font-weight: 500; transition: background .2s;}
    button:hover, .btn-modal:hover { background: #174cb8; }
    .output { margin-top: 22px; text-align: center; min-height: 26px;}
    #qrcode { margin: 19px auto 0 auto;}
    .small { font-size: 0.96em; color: #888; }
    .hidden { display: none !important; }
    .success { color: var(--success); font-weight: 500; }
    .error { color: var(--error); font-weight: 500; }
    .img-preview { margin: 10px auto 0 auto; display: block; max-width: 210px; border: 1px solid var(--border); border-radius: 6px;}
    .footer { margin: 36px auto 10px auto; text-align: center; font-size: 0.97em; color: #888;}
    .dark .footer { color: #aaa;}
    .mode-switch { float: right; margin-top: -32px; margin-bottom: 12px;}
    .mode-switch button { background: none; border: none; color: var(--primary); font-size: 1.2em; cursor: pointer; padding: 0 0 0 10px; vertical-align: middle;}
    .modal { display: none; position: fixed; z-index: 12; left: 0; top: 0; width: 100vw; height: 100vh; background: rgba(32,32,40,0.6); align-items: center; justify-content: center;}
    .modal.active { display: flex;}
    .modal-content { background: var(--panel); color: var(--text); padding: 32px 18px 22px 18px; border-radius: 12px; max-width: 90vw; width: 380px; box-shadow: 0 6px 36px #0004; text-align: center; position: relative;}
    .btn-modal { margin-top: 20px; background: var(--primary); color: #fff; border: none; border-radius: 6px; padding: 10px 30px; font-size: 1.03em; font-weight: 500; cursor: pointer; transition: background .2s;}
    .btn-modal:active { background: #174cb8;}
    #modal-decrypt-qrcode { margin: 15px auto; }
    #modal-download-btn { margin-top: 8px; }
    @media (max-width: 680px) { .container { padding: 18px 3vw; } .modal-content { width: 94vw; } }
    .footer a { color: var(--primary); text-decoration: none;}
    .footer a:hover { text-decoration: underline;}
  </style>
</head>
<body>
  <div class="container">
    <div class="mode-switch">
      <button id="darkModeBtn" title="Switch dark/light mode">🌙</button>
    </div>
    <h1>🔐 Password-Protected QR Code</h1>
    <div class="tabs">
      <div class="tab active" id="tab-generate" onclick="showTab('generate')">Generate QR</div>
      <div class="tab" id="tab-decrypt" onclick="showTab('decrypt')">Decrypt QR</div>
    </div>
    <div id="panel-generate">
      <form id="form-generate" autocomplete="off">
        <div class="form-group">
          <label for="secret">Secret Message <span class="small">(supports multiline up to ~180 chars for best reliability)</span></label>
          <textarea id="secret" rows="3" maxlength="300" style="width:100%;resize:vertical;border-radius:7px;font-size:1rem;padding:9px;" required autocomplete="off"></textarea>
        </div>
        <div class="form-group">
          <label for="password">Password <span class="small">(minimum 13 alphanumeric and special chars)</span></label>
          <input type="password" id="password" minlength="13" required>
        </div>
        <div class="form-group">
          <label for="password2">Confirm Password</label>
          <input type="password" id="password2" minlength="6" required>
        </div>
        <button type="submit">Generate QR Code</button>
      </form>
      <div class="output" id="gen-output"></div>
      <div id="qrcode"></div>
      <div id="download-btn" class="hidden">
        <button onclick="downloadQR()">⬇ Download QR Code</button>
      </div>
    </div>
    <div id="panel-decrypt" class="hidden">
      <form id="form-decrypt" autocomplete="off">
        <div class="form-group">
          <label for="qrimg">Select QR Code Image (PNG/JPG):</label>
          <input type="file" id="qrimg" accept="image/png, image/jpeg" required>
        </div>
        <div class="form-group">
          <label for="dec-password">Password</label>
          <input type="password" id="dec-password" required>
        </div>
        <button type="submit">Decrypt Message</button>
      </form>
      <div class="output" id="dec-output"></div>
      <img id="qr-preview" class="img-preview hidden" src="#" alt="QR Preview">
    </div>
  </div>
  <!-- Modal for Decrypted Message + QR -->
  <div id="modal" class="modal" onclick="closeModal(event)">
    <div class="modal-content" onclick="event.stopPropagation();">
      <div id="modal-msg" style="white-space: pre-line;word-break:break-word;"></div>
      <div id="modal-decrypt-qrcode"></div>
      <div id="modal-download-btn" class="hidden">
        <button onclick="downloadDecryptedQR()">⬇ Download QR of Decrypted</button>
      </div>
      <button class="btn-modal" onclick="closeModal()">Close</button>
    </div>
  </div>
  <div class="footer">
    Built with <span aria-label="love">❤️</span> for GitHub Pages.<br>
    <a href="https://github.com/ulyhome-live/qrcode" target="_blank">[Source code]</a>
    <span class="small"><br>No data ever leaves your browser.</span>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
  <script>
    // --- DARK MODE LOGIC ---
    function isDarkSystem() {
      return window.matchMedia('(prefers-color-scheme: dark)').matches;
    }
    function setDarkMode(enable) {
      document.documentElement.classList.toggle('dark', enable);
      localStorage.setItem('qrtool-dark', enable ? '1' : '0');
      document.getElementById('darkModeBtn').textContent = enable ? '☀️' : '🌙';
    }
    function toggleDarkMode() {
      const dark = document.documentElement.classList.contains('dark');
      setDarkMode(!dark);
    }
    (function() {
      const ls = localStorage.getItem('qrtool-dark');
      if (ls !== null) setDarkMode(ls === '1');
      else setDarkMode(isDarkSystem());
    })();
    document.getElementById('darkModeBtn').onclick = toggleDarkMode;

    // --- TAB LOGIC ---
    function showTab(tab) {
      document.getElementById('panel-generate').classList.toggle('hidden', tab !== 'generate');
      document.getElementById('panel-decrypt').classList.toggle('hidden', tab !== 'decrypt');
      document.getElementById('tab-generate').classList.toggle('active', tab === 'generate');
      document.getElementById('tab-decrypt').classList.toggle('active', tab === 'decrypt');
      document.getElementById('gen-output').textContent = '';
      document.getElementById('dec-output').textContent = '';
      document.getElementById('qrcode').innerHTML = '';
      document.getElementById('download-btn').classList.add('hidden');
      document.getElementById('qr-preview').classList.add('hidden');
      document.getElementById('form-generate').reset();
      document.getElementById('form-decrypt').reset();
    }

    // --- CRYPTO HELPERS ---
    async function getKey(password, salt, usage) {
      const enc = new TextEncoder();
      return crypto.subtle.importKey(
        'raw', enc.encode(password), {name: 'PBKDF2'}, false, ['deriveKey']
      ).then(baseKey =>
        crypto.subtle.deriveKey(
          {
            name: 'PBKDF2',
            salt,
            iterations: 100000,
            hash: 'SHA-256'
          },
          baseKey,
          { name: 'AES-CBC', length: 256 },
          false,
          usage
        )
      );
    }
    async function encryptData(secret, password) {
      const enc = new TextEncoder();
      const salt = crypto.getRandomValues(new Uint8Array(16));
      const iv = crypto.getRandomValues(new Uint8Array(16));
      const key = await getKey(password, salt, ['encrypt']);
      const data = enc.encode(secret);
      const ct = await crypto.subtle.encrypt({name: 'AES-CBC', iv}, key, data);
      // Return as base64-encoded JSON
      const payload = {
        salt: Array.from(salt),
        iv: Array.from(iv),
        ct: Array.from(new Uint8Array(ct))
      };
      return btoa(JSON.stringify(payload));
    }
    async function decryptData(b64payload, password) {
      try {
        const cleanB64 = (b64payload||"").replace(/[\n\r\s]+/g,'');
        const payload = JSON.parse(atob(cleanB64));
        const salt = new Uint8Array(payload.salt);
        const iv = new Uint8Array(payload.iv);
        const ct = new Uint8Array(payload.ct);
        const key = await getKey(password, salt, ['decrypt']);
        const pt = await crypto.subtle.decrypt({name: 'AES-CBC', iv}, key, ct);
        return new TextDecoder().decode(pt);
      } catch (e) {
        throw new Error("Decryption failed: wrong password or invalid QR code.");
      }
    }

    // --- GENERATE QR CODE ---
    document.getElementById('form-generate').onsubmit = async function(e) {
      e.preventDefault();
      const secret = document.getElementById('secret').value;
      const pw = document.getElementById('password').value;
      const pw2 = document.getElementById('password2').value;
      const output = document.getElementById('gen-output');
      output.textContent = '';
      document.getElementById('qrcode').innerHTML = '';
      document.getElementById('download-btn').classList.add('hidden');
      if (pw !== pw2) {
        output.innerHTML = '<span class="error">Passwords do not match.</span>';
        return;
      }
      if (pw.length < 6) {
        output.innerHTML = '<span class="error">Password must be at least 6 characters.</span>';
        return;
      }
      if (secret.length > 300) {
        output.innerHTML = '<span class="error">Secret message too long for QR code!</span>';
        return;
      }
      output.innerHTML = 'Encrypting...';
      try {
        const encrypted = await encryptData(secret, pw);
        if (encrypted.length > 900) {
          output.innerHTML = '<span class="error">Encrypted data too large for QR code! Try a shorter secret or password.</span>';
          return;
        }
        output.innerHTML = '<span class="success">QR code generated below!</span>';
        setTimeout(() => {
          new QRCode(document.getElementById('qrcode'), {
            text: encrypted,
            width: 300,
            height: 300,
            correctLevel: QRCode.CorrectLevel.H
          });
          document.getElementById('download-btn').classList.remove('hidden');
        }, 100);
      } catch (err) {
        output.innerHTML = '<span class="error">Error: ' + err.message + '</span>';
      }
    };
    window.downloadQR = function() {
      const qrCanvas = document.querySelector('#qrcode canvas');
      if (!qrCanvas) return;
      const link = document.createElement('a');
      link.href = qrCanvas.toDataURL();
      link.download = 'secret-qr.png';
      link.click();
    };
    document.getElementById('form-decrypt').onsubmit = async function(e) {
      e.preventDefault();
      const file = document.getElementById('qrimg').files[0];
      const password = document.getElementById('dec-password').value;
      const output = document.getElementById('dec-output');
      output.textContent = '';
      if (!file) {
        output.innerHTML = '<span class="error">Please select a QR image.</span>';
        return;
      }
      const reader = new FileReader();
      reader.onload = async function(evt) {
        const img = new window.Image();
        img.onload = async function() {
          document.getElementById('qr-preview').src = evt.target.result;
          document.getElementById('qr-preview').classList.remove('hidden');
          const canvas = document.createElement('canvas');
          canvas.width = img.width;
          canvas.height = img.height;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0);
          const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
          const code = jsQR(imageData.data, canvas.width, canvas.height);
          if (!code) {
            output.innerHTML = '<span class="error">No QR code found in image.</span>';
            return;
          }
          try {
            output.innerHTML = 'Decrypting...';
            const decrypted = await decryptData(code.data, password);
            showModal(`<span class="success">Decrypted message:</span><br><pre style="white-space: pre-wrap;font-size:1.1em;">${escapeHtml(decrypted)}</pre>`, decrypted);
            output.innerHTML = '';
          } catch (err) {
            output.innerHTML = '<span class="error">' + err.message + '</span>';
          }
        };
        img.onerror = function() {
          output.innerHTML = '<span class="error">Invalid image file.</span>';
        };
        img.src = evt.target.result;
      };
      reader.readAsDataURL(file);
    };
    function showModal(msg, plainText) {
      document.getElementById('modal-msg').innerHTML = msg;
      var modalQRDiv = document.getElementById('modal-decrypt-qrcode');
      modalQRDiv.innerHTML = '';
      document.getElementById('modal-download-btn').classList.add('hidden');
      if (plainText && plainText.length <= 2950) {
        setTimeout(function() {
          new QRCode(modalQRDiv, {
            text: plainText,
            width: 200,
            height: 200,
            correctLevel: QRCode.CorrectLevel.H
          });
          document.getElementById('modal-download-btn').classList.remove('hidden');
        }, 100);
      }
      document.getElementById('modal').classList.add('active');
    }
    function downloadDecryptedQR() {
      var qrCanvas = document.querySelector('#modal-decrypt-qrcode canvas');
      if (!qrCanvas) return;
      var link = document.createElement('a');
      link.href = qrCanvas.toDataURL();
      link.download = 'decrypted-message-qr.png';
      link.click();
    }
    function closeModal(evt) { document.getElementById('modal').classList.remove('active'); }
    function escapeHtml(str) {
      return (str||'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
    }
  </script>
</body>
</html>
```

---

# 📝 **How to Use and Deploy**

### **1. Save this code as `index.html` in your GitHub repo.**

### **2. Commit and push to your repository.**

### **3. Enable GitHub Pages:**

* Go to your repo settings → "Pages"
* Select your main branch (and `/root` if asked)
* After a minute, your site will be live at
  `https://your-username.github.io/your-repo/`

### **4. Use the app!**

* Open your GitHub Pages URL
* Use the tabs to **generate** or **decrypt** password-protected QR codes—all in the browser.
* **No need for Linux or Codespaces, works everywhere—even on phones!**

---

## ✅ Here's the python version that will work locally for QRCode only!

---

This version includes:

* A **Graphical User Interface (GUI)** using `tkinter`.
* Ability to generate QR codes for:

  1. **Plain Text / URLs**
  2. **Wi-Fi Credentials**
  3. **SMS Messages**
* Automatically installs missing dependencies.

---

## 🐍 Full Updated Python Script (Cross-Platform GUI Tool)

> 💡 **Save this as `qr_generator_gui.py`** and delete any conflicting files like `qrcode.py` first!

```python
import subprocess
import sys
import os

# Step 1: Ensure dependencies are installed
def install_dependencies():
    try:
        import qrcode
        from PIL import Image, ImageTk
        import tkinter as tk
        from tkinter import ttk, messagebox
    except ImportError:
        print("Installing required packages...")
        subprocess.check_call([sys.executable, "-m", "pip", "install", "qrcode", "pillow"])
        import qrcode
        from PIL import Image, ImageTk
        import tkinter as tk
        from tkinter import ttk, messagebox
    return qrcode, Image, ImageTk, tk, ttk, messagebox

qrcode_lib, Image, ImageTk, tk, ttk, messagebox = install_dependencies()


# Step 2: Define QR generation logic
def generate_qr(content, output_path="qrcode_output.png"):
    qr = qrcode_lib.QRCode(
        version=1,
        error_correction=qrcode_lib.constants.ERROR_CORRECT_Q,
        box_size=10,
        border=4
    )
    qr.add_data(content)
    qr.make(fit=True)
    img = qr.make_image(fill_color="black", back_color="white")
    img.save(output_path)
    return output_path


# Step 3: GUI logic
def launch_gui():
    def clear_fields():
        for widget in input_frame.winfo_children():
            widget.grid_remove()
        for widget in input_entries:
            widget.delete(0, tk.END)

    def show_fields(selection):
        clear_fields()
        if selection == "Text / URL":
            lbl = ttk.Label(input_frame, text="Enter Text or URL:")
            lbl.grid(row=0, column=0, sticky="w")
            entry = ttk.Entry(input_frame, width=40)
            entry.grid(row=0, column=1)
            input_entries[:] = [entry]
        elif selection == "Wi-Fi":
            labels = ["SSID:", "Password:", "Security (WPA/WEP/n):"]
            for i, label in enumerate(labels):
                ttk.Label(input_frame, text=label).grid(row=i, column=0, sticky="w")
                entry = ttk.Entry(input_frame, width=30)
                entry.grid(row=i, column=1)
                input_entries.append(entry)
        elif selection == "SMS":
            ttk.Label(input_frame, text="Phone Number:").grid(row=0, column=0, sticky="w")
            phone_entry = ttk.Entry(input_frame, width=30)
            phone_entry.grid(row=0, column=1)

            ttk.Label(input_frame, text="Message:").grid(row=1, column=0, sticky="w")
            message_entry = ttk.Entry(input_frame, width=30)
            message_entry.grid(row=1, column=1)

            input_entries[:] = [phone_entry, message_entry]

    def on_generate():
        selection = type_var.get()
        if selection == "Text / URL":
            text = input_entries[0].get().strip()
            if not text:
                messagebox.showerror("Error", "Text cannot be empty.")
                return
            content = text

        elif selection == "Wi-Fi":
            ssid = input_entries[0].get().strip()
            password = input_entries[1].get().strip()
            security = input_entries[2].get().strip().upper() or "WPA"
            if not ssid:
                messagebox.showerror("Error", "SSID is required.")
                return
            content = f"WIFI:T:{security};S:{ssid};P:{password};;"

        elif selection == "SMS":
            number = input_entries[0].get().strip()
            msg = input_entries[1].get().strip()
            if not number:
                messagebox.showerror("Error", "Phone number is required.")
                return
            content = f"SMSTO:{number}:{msg}"

        else:
            messagebox.showerror("Error", "Please select a type.")
            return

        try:
            output_path = generate_qr(content)
            img = Image.open(output_path).resize((300, 300), Image.LANCZOS)
            photo = ImageTk.PhotoImage(img)
            qr_label.config(image=photo)
            qr_label.image = photo
        except Exception as e:
            messagebox.showerror("Error", f"Failed to generate QR Code:\n{e}")

    # GUI Window
    root = tk.Tk()
    root.title("QR Code Generator")
    root.geometry("500x500")
    root.resizable(False, False)

    # Top selection area
    type_var = tk.StringVar()
    ttk.Label(root, text="Choose QR Code Type:").pack(pady=(10, 0))
    combo = ttk.Combobox(root, textvariable=type_var, values=["Text / URL", "Wi-Fi", "SMS"], state="readonly", width=20)
    combo.pack(pady=(0, 10))
    combo.bind("<<ComboboxSelected>>", lambda e: show_fields(type_var.get()))

    # Frame for dynamic input fields
    input_frame = ttk.Frame(root)
    input_frame.pack(pady=10)

    input_entries = []  # dynamic entry fields

    # Generate button
    ttk.Button(root, text="Generate QR Code", command=on_generate).pack(pady=10)

    # Label for displaying QR code
    qr_label = ttk.Label(root)
    qr_label.pack(pady=10)

    root.mainloop()


# Step 4: Run GUI
if __name__ == "__main__":
    launch_gui()
```

---

## 🧪 How to Use

### 1. **Save as**: `qr_generator_gui.py`

> 🚫 Make sure there's **no file named `qrcode.py`** in the same folder!

### 2. **Run the script**:

```bash
python qr_generator_gui.py
```

### 3. **Pick your mode**:

* Select one of:

  * "Text / URL"
  * "Wi-Fi"
  * "SMS"
* Fill the inputs.
* Click **"Generate QR Code"** — it will display the QR code instantly.

---

## 🧩 Dependencies (auto-installed)

| Package   | Purpose                      |
| --------- | ---------------------------- |
| `qrcode`  | QR Code generation           |
| `Pillow`  | Image rendering for GUI      |
| `tkinter` | Cross-platform GUI (builtin) |

---


