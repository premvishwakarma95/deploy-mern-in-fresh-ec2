# How to deploy Mern in fresh EC2 instance with Linux OS

## 1️⃣ Connect to EC2
Command to connect ec2 instance using sssh remember os should be ubuntu only then this works because we are using ubuntu as a user.
```bash
ssh -i your-key.pem ubuntu@<EC2_PUBLIC_IP>
```

---

## 2️⃣ Update the system (FIRST command always)
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 3️⃣ Install Node.js (recommended: NodeSource)
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```
- Verify
```bash
node -v
npm -v
```

---

## 4️⃣ Install Git
- First by `git -v` this command if exists then don't don't this command if not then insatll.
```basah
sudo apt install git -y
```

## 5️⃣ Clone your project
```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

---

## 6️⃣ Install dependencies
```bash
npm install
```

---

## 7️⃣ Set environment variables
- Option A: .env file
```bash
nano .env
```
```env
PORT=3000
DB_URL=your_db_url
NODE_ENV=production
```
