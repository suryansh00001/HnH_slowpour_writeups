I don't know how to read backend script actually
perplexity helped a bit in reading and understanding the code




    1. Password Generation and Protection
    a. Wordlist Source
    Passwords are selected randomly from rockyou.txt (split into 20 parts, e.g., rockyou_part_1.txt to rockyou_part_20.txt).
    
    b. Encoding/Encryption by Level
    Easy ("eazy")
    Three random reversible encodings are applied in sequence, chosen from:
    
    Base64 (Bas64)
    
    Hex (Hez)
    
    Base32 (Bas32)
    
    Base16 (Bas16)
    
    ROT13 (ROT_13)
    
    Each encoding step is logged and can be reversed if the order is known.
    
    Medium ("medum")
    First step: Irreversible hash (MD5 or SHA1) or string reversal.
    
    Next two steps: Random reversible encodings from the Easy list or string reversal.
    
    Hashing makes direct reversal impossible; only brute-force or rainbow tables can help.
    
    Hard ("hardy")
    Password + salt (epoch time) → SHA256 hash.
    
    Not reversible; only brute-force with the correct salt can crack it.
    
    2. Decoding and Cracking Approach
    a. Easy Level
    Obtain the Encoded Password and Steps:
    
    If the encoding steps are known (from cookies or metadata), apply them in reverse.
    
    If unknown, brute-force all possible sequences (125 combinations).
    
    Reverse Each Encoding:
    
    For each possible combination, decode the password.
    
    Check if the result is a readable string and exists in rockyou.txt.
    
    b. Medium Level
    Identify the Hash:
    
    If the first step is a hash (MD5/SHA1), direct reversal is impossible.
    
    Use brute-force: hash each rockyou.txt password and compare.
    
    Apply Remaining Encodings:
    
    If there are further reversible steps, apply them as per the recorded order.
    
    c. Hard Level
    SHA256 with Salt:
    
    Password is concatenated with the epoch time (salt) and hashed.
    
    Brute-force by combining each rockyou.txt password with the salt and hashing.
    
    3. Script Analysis and Exploitation
    a. Backend Script Insights
    The script uses Python's base64, codecs, and hashlib libraries for encoding and hashing.
    
    Encodings are applied via randomly chosen functions; the order is critical for decoding.
    
    For each challenge, the metadata (encoding steps, salt, etc.) is stored in cookies, sometimes base64-encoded.
    
    b. Exfiltration and Automation
    Automate decoding: Write a script to try all encoding combinations for Easy, hash and compare for Medium, and hash with salt for Hard.
    
    Avoid bans: Limit attempts, use proxies, or analyze cookies locally to minimize server requests.
    
    4. Example Python Approach
    python
    import base64, codecs, hashlib, itertools
    
    def encode_step(val, method):
        if method == "Bas64":
            return base64.b64encode(val.encode()).decode()
        elif method == "Hez":
            return val.encode().hex()
        elif method == "Bas32":
            return base64.b32encode(val.encode()).decode()
        elif method == "Bas16":
            return base64.b16encode(val.encode()).decode()
        elif method == "ROT_13":
            return codecs.encode(val, 'rot_13')
        else:
            raise ValueError("Unknown method")
    
    def decode_step(val, method):
        if method == "Bas64":
            return base64.b64decode(val.encode()).decode()
        elif method == "Hez":
            return bytes.fromhex(val).decode()
        elif method == "Bas32":
            return base64.b32decode(val.encode()).decode()
        elif method == "Bas16":
            return base64.b16decode(val.encode()).decode()
        elif method == "ROT_13":
            return codecs.decode(val, 'rot_13')
        else:
            raise ValueError("Unknown method")
    Use all permutations for Easy, hash and compare for Medium, and hash with salt for Hard.
    
    5. Flag Extraction Strategy
    Capture cookies/metadata for encoding steps and salt.
    
    Automate decoding for each level as described.
    
    Submit correct passwords in sequence to progress through Easy, Medium, and Hard.
    
    Upon solving Hard, the server returns the flag in the response.






so now I get to know that I have to use cookies and python script to solve this challenge
I tried passing easy level with chat gpt decode the flag very fast for me with metadata and userdata cookie

so I generated the code to fasten the process


    import base64, hashlib, codecs, json
    
    # ------------------------------
    # Reversible Decoding Functions
    # ------------------------------
    def decode(data, method):
        try:
            if method.lower() in ["base64", "bas64"]:
                return base64.b64decode(data).decode('utf-8', errors='replace')
            elif method.lower() in ["base32", "bas32"]:
                return base64.b32decode(data, casefold=True).decode('utf-8', errors='replace')
            elif method.lower() in ["base16", "bas16", "hex", "hez"]:
                return base64.b16decode(data.upper()).decode('utf-8', errors='replace')
            elif method.lower() in ["rot13", "rot_13"]:
                return codecs.decode(data, 'rot_13')
            elif method.lower() == "reverse":
                return data[::-1]
            else:
                return "[Unsupported]"
        except Exception as e:
            return f"[Fail @ {method}: {str(e)}]"
    
    # ------------------------------
    # Load Rockyou Wordlist (Colab Path)
    # ------------------------------
    def load_wordlist(limit=None):
        path = "/content/sample_data/rockyou.txt"
        words = set()
        with open(path, "r", encoding="latin-1", errors="ignore") as f:
            for i, line in enumerate(f):
                if limit and i >= limit:
                    break
                word = line.strip()
                if word:
                    words.add(word)
        return words
    
    # ------------------------------
    # Easy Level Cracker
    # ------------------------------
    def crack_easy(enc_pwd, steps, rockyou):
        print(f"[*] Cracking Easy | Steps: {steps}")
        try:
            data = enc_pwd
            for step in reversed(steps):
                data = decode(data, step)
            if data in rockyou:
                print(f"[+] Cracked Easy Password: {data}")
            else:
                print("[!] Decoded, but not found in wordlist:", data)
        except Exception as e:
            print("[x] Error:", str(e))
    
    # ------------------------------
    # Medium Level Cracker
    # ------------------------------
    def crack_medium(enc_pwd, steps, rockyou):
        print(f"[*] Cracking Medium | Steps: {steps}")
        hash_algo = steps[0].lower()
        for word in rockyou:
            candidate = word
            # Step 1: Hash
            if hash_algo == "md5":
                hashed = hashlib.md5(candidate.encode()).hexdigest()
            elif hash_algo == "sha1":
                hashed = hashlib.sha1(candidate.encode()).hexdigest()
            else:
                continue
    
            if hashed == enc_pwd:
                print(f"[+] Cracked Hash: {word}")
                return
    
        print("[-] Hash not found in wordlist.")
    
    # ------------------------------
    # Hard Level Cracker
    # ------------------------------
    def crack_hard(enc_pwd, salt, rockyou):
        print(f"[*] Cracking Hard | Salt: {salt}")
        for word in rockyou:
            combo = word + str(salt)
            hashed = hashlib.sha256(combo.encode()).hexdigest()
            if hashed == enc_pwd:
                print(f"[+] Cracked Hard Password: {word}")
                return
        print("[-] Hash+salt not found in wordlist.")
    
    # ------------------------------
    # Main Controller
    # ------------------------------
    def main():
        level = input("Level (easy / medium / hard): ").strip().lower()
        enc_pwd = input("Encoded Password: ").strip()
        meta_cookie = input("Metadata Cookie (base64): ").strip()
    
        try:
            meta = json.loads(base64.b64decode(meta_cookie).decode())
            steps = meta.get("encryption_steps", [])
            salt = meta.get("start_time", None)
        except Exception as e:
            print("[x] Error decoding metadata:", str(e))
            return
    
        print("[*] Loading wordlist...")
        rockyou = load_wordlist()  # Or load_wordlist(limit=1000) to speed up
    
        if level == "easy":
            crack_easy(enc_pwd, steps, rockyou)
        elif level == "medium":
            crack_medium(enc_pwd, steps, rockyou)
        elif level == "hard":
            if not salt:
                print("[x] No salt (start_time) found in metadata.")
            else:
                crack_hard(enc_pwd, salt, rockyou)
        else:
            print("[x] Invalid level.")
    
    if __name__ == "__main__":
        main()


this helped me to clear all the three levels easy>medium>hard
flag was displayed after clearing all the levels
