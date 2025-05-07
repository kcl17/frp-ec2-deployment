# frp-ec2-deployment

This README provides detailed steps on how to set up and configure the **FRP server** on an **EC2 instance** running **Ubuntu**.

---

## **1. Prerequisites**

- **AWS EC2 instance** (named **FRP Instance**) running **Ubuntu**.
- **FRP (Fast Reverse Proxy)** binary files downloaded from [GitHub Releases](https://github.com/fatedier/frp/releases).
- **Open Ports:** Ensure the security group associated with the EC2 instance has the necessary ports open (e.g., port 7000 for the FRP server).
- Basic knowledge of **Linux** command line.

##

### **Step 1: Create an EC2 Instance (FRP Instance)**

1. **Login to AWS Management Console**.
2. Navigate to **EC2** and click **Launch Instance**.
3. Select the **Ubuntu** AMI (Amazon Machine Image) of your preferred version.
4. Choose the instance type (e.g., `t2.micro` for free-tier).
5. Configure the instance details as per your requirements.

---

### **Step 2: Edit Security Group to Add Port 7000**

If you need to manually edit the security group after creating the instance:

1. In the **EC2 Dashboard**, go to **Security Groups** under the **Network & Security** section.
2. Select the security group associated with your **FRP Instance**.
3. Under the **Inbound rules** tab, click **Edit inbound rules**.
4. Add a new rule:
   - **Type**: Custom TCP Rule
   - **Port Range**: 7000
   - **Source**: Anywhere (`0.0.0.0/0`) or restrict to specific IP addresses.
5. Add another rule:
   - **Type**: Custom TCP Rule
   - **Port Range**: 7500
   - **Source**: Anywhere (`0.0.0.0/0`) or restrict to specific IP addresses. 
7. Click **Save rules**.

To continue the process, you'll need to connect to your EC2 instance via SSH using the terminal. Here’s how you can modify the README to include this step:

---

### **Step 3: Connect to the EC2 Instance via PuTTY**

1. **Download PuTTY**:  
   If you don't have PuTTY installed on your machine, download it from [here](https://www.putty.org/) and install it.

2. **Convert PEM to PPK**:
   PuTTY does not support `.pem` key files directly, so you need to convert the `.pem` file to the **PPK** (PuTTY Private Key) format.

   a. Open **PuTTYgen** (which is installed along with PuTTY).

   b. In **PuTTYgen**, click **Load** and select your `.pem` file.

   c. Once loaded, click **Save private key** to save the key in the `.ppk` format.

3. **Find the Public IP Address** of your EC2 instance:  
   You can find the public IP address of your EC2 instance in the **EC2 Dashboard** under **Instances**.

4. **Launch PuTTY**:

   a. Open **PuTTY** on your machine.

   b. In the **Host Name (or IP address)** field, enter the **Public IP Address** of your EC2 instance.

   c. In the **Connection Type** section, ensure that **SSH** is selected (which is the default).

   d. Under **Category**, navigate to **Connection > SSH > Auth**.

   e. Click **Browse** and select the `.ppk` file you generated earlier.

5. **Connect to the EC2 Instance**:  
   Now, click **Open** to start the SSH session. The first time you connect, you’ll be asked to confirm the security fingerprint. Click **Yes** to continue.

6. **Login to the Instance**:  
   You should be prompted to login. For an Ubuntu instance, the default username is **`ubuntu`**. So, type `ubuntu` and hit **Enter**.

##

Once connected, you can proceed with the steps for setting up the FRP server!

---

## **2. Setup FRP Server on EC2**

### **Step 1: Install Dependencies**

Ensure your EC2 instance has the necessary tools installed. You may need `wget` or `curl` to download files:

```bash
sudo apt update
sudo apt install wget curl -y
```
### **Step 2: Download and Extract FRP**

1. Download the **FRP** binary from the official GitHub releases page:

   ```bash
   wget https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz
   ```

2. Extract the tarball:

   ```bash
   tar -zxvf frp_0.47.0_linux_amd64.tar.gz
   ```
3. Navigate to the extracted folder:

   ```bash
   cd frp_0.47.0_linux_amd64
   ```

You should now have the following files:

- `frps` (FRP server binary)
- `frps.ini` (FRP server configuration file)

---

### **Step 3: Move FRP Files to a Proper Directory**

Move the `frps` binary and the `frps.ini` configuration file to a proper directory such as `/opt/frp/`:

```bash
sudo mkdir -p /opt/frp
sudo mv frps frps.ini /opt/frp/
```
---

### **Step 4: Make the FRP Binary Executable**

Ensure the `frps` binary has executable permissions:

```bash
sudo chmod +x /opt/frp/frps
```

---

## **3. Configure the FRP Server**

You will need to modify the `frps.ini` configuration file to customize the server settings. Open the `frps.ini` file:

```bash
sudo nano /opt/frp/frps.ini
```

Modify the following common settings:

- **Bind Port** (default `7000`): This is the port where the FRP server will listen for incoming connections from the client.
- **Dashboard** (optional): If you want to enable the FRP dashboard, add or modify the following settings:

```ini
[common]
bind_port = 7000          # Port for FRP to listen to
vhost_http_port = 80      # Port for HTTP requests (optional)
vhost_https_port = 443    # Port for HTTPS requests (optional)
dashboard_port = 7500     # Port for the FRP dashboard (optional)
dashboard_user = admin    # Dashboard username (optional)
dashboard_pwd = admin     # Dashboard password (optional)

```
Save and close the file (`CTRL + X`, `Y`, `ENTER`).

## **4. Start the FRP server**

Now you can start the FRP server:

```bash
   sudo ./frps -c ./frps.ini

```
---

## **5. Create a systemd Service to Run FRP Server**

To manage the FRP server as a system service, create a systemd service file.

1. Move the extracted files to /opt/frp/ :

   ```bash
   sudo mv frp_0.47.0_linux_amd64 /opt/frp/
   ```

1. Create and edit the service file:

   ```bash
   sudo nano /etc/systemd/system/frps.service
   ```

1. Add the following content to the file:

```ini
[Unit]
Description=FRP server
After=network.target

[Service]
ExecStart=/opt/frp/frps -c /opt/frp/frps.ini
Restart=always
User=nobody
RestartSec=3

[Install]
WantedBy=multi-user.target

```
This configuration ensures that the FRP server starts on boot and restarts automatically if it crashes.

---

### **Step 5: Reload systemd and Start FRP Server**

Now, reload systemd to recognize the new service and start the FRP server:

```bash
sudo systemctl daemon-reload
sudo systemctl start frps
sudo systemctl enable frps
```
This will start the FRP server and ensure it restarts on system reboots.

---

### **Step 6: Verify the Service Status**

Check the status of the FRP service to ensure it's running properly:

```bash
sudo systemctl status frps
```

The status should show `active (running)`.
---

## **6. Enable FRP Dashboard**

If you've enabled the FRP dashboard (via `dashboard_port = 7500` in `frps.ini`), you can access it in your browser:

```
http://<your-ec2-public-ip>:7500
```

Login using the credentials set in `frps.ini` (default: `admin` / `admin`).
---

## **7. Conclusion**

Your FRP server is now set up and running on your EC2 instance. You can use FRP to forward ports, access internal services, and more!

For more advanced configurations, refer to the [FRP documentation](https://github.com/fatedier/frp/blob/master/README.md).
