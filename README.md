# miniproject-on-SGs-and-NACLs

---

# 🔐 **AWS Security Groups vs Network ACLs – Detailed Guide**

---

## 1️⃣ **What Are Security Groups?**

**Security Groups (SGs)** are **virtual firewalls** for **EC2 instances** and other AWS resources like RDS databases or Elastic Load Balancers.

They **control inbound and outbound traffic** *at the instance level*.

### 🛠️ **Key Features:**

* **Instance-level firewall** (applied directly to ENIs — Elastic Network Interfaces)
* **Stateful** – If traffic is allowed in, the return traffic is automatically allowed out (and vice versa).
* **Allow rules only** – You cannot create “deny” rules.
* Supports rules by:

  * **IP range** (CIDR block)
  * **Another security group**
  * **Port and protocol**
* You can attach multiple security groups to a single resource.

---

### 📥 **Inbound Rules Example**

| Type  | Protocol | Port Range | Source           |
| ----- | -------- | ---------- | ---------------- |
| SSH   | TCP      | 22         | 203.0.113.0/24   |
| HTTP  | TCP      | 80         | 0.0.0.0/0        |
| HTTPS | TCP      | 443        | MyLoadBalancerSG |

---

### 📤 **Outbound Rules Example**

| Type | Protocol | Port Range | Destination |
| ---- | -------- | ---------- | ----------- |
| All  | All      | All        | 0.0.0.0/0   |

---

📌 **Best Practices for SGs:**

* Use **least privilege** (only open the ports and IPs you need).
* Use **separate SGs** for different roles (e.g., web server SG, DB SG).
* Reference **SGs instead of IPs** when linking resources (e.g., allow only “WebServerSG” to talk to “DatabaseSG”).

---

## 2️⃣ **What Are Network ACLs (NACLs)?**

**Network Access Control Lists** are **stateless firewalls** that operate **at the subnet level** in a VPC.

They control **inbound and outbound traffic to and from entire subnets**.

---

### 🛠️ **Key Features:**

* **Subnet-level firewall** – affects all resources in that subnet.
* **Stateless** – Return traffic must be explicitly allowed by outbound rules.
* Can have **allow** and **deny** rules.
* Rules are **evaluated in order** from the lowest rule number to the highest.
* Default NACL allows all traffic in and out unless modified.

---

### 📥 **Inbound NACL Rule Example**

| Rule # | Allow/Deny | Protocol | Port Range | Source    |
| ------ | ---------- | -------- | ---------- | --------- |
| 100    | ALLOW      | TCP      | 80         | 0.0.0.0/0 |
| 110    | DENY       | TCP      | 22         | 0.0.0.0/0 |
| \*     | DENY       | All      | All        | All       |

---

### 📤 **Outbound NACL Rule Example**

| Rule # | Allow/Deny | Protocol | Port Range | Destination |
| ------ | ---------- | -------- | ---------- | ----------- |
| 100    | ALLOW      | TCP      | 1024-65535 | 0.0.0.0/0   |
| \*     | DENY       | All      | All        | All         |

---

📌 **Best Practices for NACLs:**

* Use NACLs to **block specific traffic** at the subnet level (e.g., known malicious IP ranges).
* Always have a **catch-all DENY rule** at the end (implicit anyway).
* For public subnets, allow inbound HTTP/HTTPS and ephemeral ports for return traffic.

---

## 3️⃣ **Security Groups vs NACLs – Side-by-Side Comparison**

| Feature              | Security Group                       | Network ACL                                 |
| -------------------- | ------------------------------------ | ------------------------------------------- |
| **Scope**            | Instance level (ENI)                 | Subnet level                                |
| **Stateful?**        | ✅ Yes                                | ❌ No (stateless)                            |
| **Rules Type**       | Allow only                           | Allow & Deny                                |
| **Default Behavior** | Deny all inbound, allow all outbound | Allow all inbound & outbound (default NACL) |
| **Evaluation Order** | All rules evaluated before decision  | Rules evaluated in order by rule number     |
| **Typical Use**      | Control specific resource access     | Add extra layer of subnet-level filtering   |

---

## 4️⃣ **How They Work Together in AWS**

* **Traffic to an EC2 instance** must pass **both** the NACL for the subnet and the Security Group(s) attached to the instance.
* If either blocks the traffic, it’s denied.
* SGs are **first line of defense at the instance level**, NACLs provide **network-wide filtering**.

---

## 5️⃣ **Real-World AWS Example**

Imagine you have:

* **Public subnet** for web servers
* **Private subnet** for databases

**Security Groups:**

* WebServerSG allows inbound HTTP/HTTPS from anywhere and outbound to DBServerSG on port 3306.
* DBServerSG allows inbound from WebServerSG only.

**NACLs:**

* PublicSubnetNACL allows inbound HTTP/HTTPS from 0.0.0.0/0, denies all other inbound.
* PrivateSubnetNACL allows inbound MySQL (3306) only from PublicSubnet's CIDR.

This **defense-in-depth** approach means:

1. Even if a SG is misconfigured, NACL provides a backup.
2. Attacks from malicious IPs can be blocked subnet-wide.

---

To demonstarate this concepts, the mini project would be carried out in two parts

-----------------------------------------------
PART 1
a) An instance my-test-instance was launched and a web application was transfered into the appropriate location to be served by apache. The security group for the instance allows only inbound ssh traffic.
<img width="947" height="389" alt="image" src="https://github.com/user-attachments/assets/a080731f-ee51-48dd-8fa4-74bc4792917f" />
<img width="955" height="411" alt="image" src="https://github.com/user-attachments/assets/7643f1fc-5054-4c56-8ffb-7aba0f848437" />
<img width="952" height="381" alt="image" src="https://github.com/user-attachments/assets/5f3ecf64-e745-47f1-b2f1-c97b41ad13f7" />
<img width="952" height="381" alt="image" src="https://github.com/user-attachments/assets/281b3239-e6a6-463d-ba38-e462f0a3fe06" />
<img width="948" height="393" alt="image" src="https://github.com/user-attachments/assets/6153a4db-3bac-45b2-8f2f-6a7b4d5eb83e" />
b) Upon retreving the public IP address and opening, the web app coundnt be reached
<img width="953" height="400" alt="image" src="https://github.com/user-attachments/assets/9415acc9-ac63-496e-b8a6-37abb9ebc9d4" />
<img width="959" height="473" alt="image" src="https://github.com/user-attachments/assets/9012a7b2-edf9-46d6-ae30-e7a706b02a02" />
c) Edit or create a security group and allow http on port 80.
<img width="951" height="389" alt="image" src="https://github.com/user-attachments/assets/748e9d54-6737-4aa6-82c6-6caa9ea4d6e3" />
<img width="941" height="341" alt="image" src="https://github.com/user-attachments/assets/780eccc3-8c68-4ac6-807e-43a69c54567a" />
<img width="946" height="381" alt="image" src="https://github.com/user-attachments/assets/e57cc339-a4f7-4149-9107-5eae9d5c536f" />
d) Reload the public IP
<img width="959" height="479" alt="image" src="https://github.com/user-attachments/assets/946f0966-b2a8-4193-b208-8781308ba281" />

e) Lets see what happens when we edit the outbound rules 
<img width="941" height="389" alt="image" src="https://github.com/user-attachments/assets/5d5c289e-b79a-4724-95c3-09aa189bdec2" />
<img width="951" height="394" alt="image" src="https://github.com/user-attachments/assets/4306c248-2577-4d2f-81d7-ee80fca525c9" />
<img width="959" height="379" alt="image" src="https://github.com/user-attachments/assets/1c7ce3b8-7307-44f8-a478-63fa01fda6c5" />
<img width="955" height="404" alt="image" src="https://github.com/user-attachments/assets/f9840bca-b2de-4593-994e-4914d1d4e33a" />
<img width="956" height="486" alt="image" src="https://github.com/user-attachments/assets/3dbc132b-44a8-4db8-9536-80397b5cd6ae" />
This clearly demonstrates the stateful property of security groups.

f) If we delete both inbound and outbound rules, lets see if we would reach the web app.
<img width="946" height="383" alt="image" src="https://github.com/user-attachments/assets/e10ee495-d417-42c9-92fb-942758937ebb" />
<img width="950" height="385" alt="image" src="https://github.com/user-attachments/assets/42ff0c4f-5e11-414d-bab5-6a77a8cfa6fa" />
<img width="955" height="404" alt="image" src="https://github.com/user-attachments/assets/f9840bca-b2de-4593-994e-4914d1d4e33a" />
<img width="940" height="464" alt="image" src="https://github.com/user-attachments/assets/c99638ec-cea3-4b68-9837-fcd2b3afe642" />
The web page times out. 

Now, let us edit the outbound rules and allow http on port 80.
<img width="952" height="299" alt="image" src="https://github.com/user-attachments/assets/40cd57b1-1133-4199-9fd2-75d913b16280" />
<img width="937" height="373" alt="image" src="https://github.com/user-attachments/assets/1884c765-f259-4052-ab82-576d48c81f0a" />
<img width="959" height="496" alt="image" src="https://github.com/user-attachments/assets/8700d505-c6ca-4b77-ac0c-7153cf5704c1" />
but there is connectivity with the local host
<img width="959" height="431" alt="image" src="https://github.com/user-attachments/assets/e35768bd-9bd9-4df7-8021-c0fed0d2f630" />

PART 2

a) navigate to vpc and click on NACLs
<img width="957" height="436" alt="image" src="https://github.com/user-attachments/assets/c67c7c68-8bbc-49fa-b422-c3f4bea9ec79" />
<img width="950" height="391" alt="image" src="https://github.com/user-attachments/assets/71036da2-7fda-454e-96ac-480b1ef1d88c" />
<img width="936" height="373" alt="image" src="https://github.com/user-attachments/assets/3bd458ad-aaa4-416a-924e-d7627e8141b3" />
<img width="928" height="382" alt="image" src="https://github.com/user-attachments/assets/8f18007f-f4a0-40c5-b1aa-ce2aba2181a0" />
both inbound and outbound allow traffic from anywhere IPv4. Hence, the web app displays
<img width="956" height="479" alt="image" src="https://github.com/user-attachments/assets/826499da-073a-4abb-8ee0-21eb398abe3c" />

b) now lets edit only the outbound and check again
<img width="943" height="422" alt="image" src="https://github.com/user-attachments/assets/58bc14a8-e713-48a2-a64c-5532a8335ec7" />

<img width="949" height="394" alt="image" src="https://github.com/user-attachments/assets/9dbc0fdb-715c-439a-9856-8f7f3117b5e2" />

<img width="950" height="481" alt="image" src="https://github.com/user-attachments/assets/b0df1f88-15ed-4782-8e86-6eacfd7cf008" />

this reinforces how stateless NACLs are.












