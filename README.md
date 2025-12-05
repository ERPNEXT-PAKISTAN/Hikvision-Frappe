# Hikvision-Frappe
Connect Hikvision Devices with ERPNext

<p align="center">
  <img src="https://seeklogo.com/images/H/hikvision-logo-647E63441B-seeklogo.com.png" alt="Hikvision Logo" width="150" />
</p>

<h1 align="center">Hikvision x ERPNext Integration Guide</h1>

---

## ✅ 1. Does Our Method Support All Hikvision Models?

**Short answer:** <span style="color:red"><strong>NO</strong></span> — because Hikvision has **3 different families** of devices, and they do **NOT** all support the same APIs.

---

### 🔵 Hikvision Device Types & API Capabilities

<table>
<tr>
  <th>Type</th>
  <th>Model Examples</th>
  <th>Supported Features</th>
  <th>Limitations</th>
</tr>
<tr>
  <td><b>TYPE A</b><br/>Access Control Panels / Face Terminals</td>
  <td><b>DS-K1Txxxx series</b></td>
  <td>
    <ul>
      <li>✅ Real-time event streaming
        <br/><code>/ISAPI/Event/notification/alertStream</code>
      </li>
    </ul>
  </td>
  <td>
    <ul>
      <li>❌ No historical logs retrieval</li>
      <li>❌ GET requests for past attendance:
        <br/><code>/ISAPI/AccessControl/AcsEvent?format=json</code>
        <br/><code>/ISAPI/Attendance/Record</code>
      </li>
      <li>→ Returns <span style="color:red">"Invalid Operation"</span> / <span style="color:red">"notSupport"</span></li>
    </ul>
  </td>
</tr>
<tr>
  <td><b>TYPE B</b><br/>Old DVR/NVR-based Access Models</td>
  <td>-</td>
  <td>
    <ul>
      <li>✅ Some event pull</li>
    </ul>
  </td>
  <td>
    <ul>
      <li>❌ Very inconsistent</li>
      <li>❌ Not standardized</li>
    </ul>
  </td>
</tr>
<tr>
  <td><b>TYPE C</b><br/>OEM Fingerprint Devices (ZKTeco engine)</td>
  <td>-</td>
  <td>
    <ul>
      <li>✅ Support full history</li>
      <li>✅ Similar to ZKTeco API</li>
    </ul>
  </td>
  <td>
    <ul>
      <li>❌ Not used in your model</li>
    </ul>
  </td>
</tr>
</table>

---

### 🟥 The Reality for Your Device Model

Your device **ONLY** supports **REAL-TIME push** via `alertStream`—not historical retrieval.

This is **confirmed** by your `curl` results.

---

#### ❗ IMPORTANT CONSEQUENCE

If the device is offline for:
- 1 day
- 1 week
- 1 month

→ It **CANNOT** send past events again.

> **This is a HARD LIMITATION from Hikvision firmware.**
>
> Not ERPNext.  
> Not our app.  
> Not Python.  
> The device does not store event history for retrieval.

So all your GET requests return:
```json
{
  "statusString": "Invalid Operation",
  "subStatusCode": "notSupport"
}
```

---

## 🟩 HOW COMPANIES SOLVE THIS? TWO WAYS

| Option | Description | Recommended for |
| ------ | ----------- | --------------- |
| <b>1️⃣ Real-Time Only (Recommended)</b> | Permanent TCP streaming listener | Stable networks |
| <b>2️⃣ ERP Push (Webhook Mode)</b> | Events sent to a server URL (Event Subscription) | Models supporting ISAPI Event Subscription |

---

## 🟦 What You Want: <span style="color:green"><strong>100% Past + Real-time Sync</strong></span>

Possible **only if:**
- ✔ Device supports event subscription
- ✔ Device buffers offline events internally

*Not all Hikvision models support this.*

---

## 🟪 SO, CONFIRM THIS ONE FINAL THING

Go to your Hikvision device web interface:

```
Configuration → Network → Advanced → Event Notification / Alarm Host
```

Look for options like:

- ✔ **"Alarm Host"** (IP + Port)
- ✔ **"Event Subscription"**
- ✔ **"HTTP Notification" / "Webhook"**
- ✔ **"Upload Offline Events"**

**Example screenshot for reference:**
```
Event Streaming: Enable
Event Buffering: Enable
HTTP Event Receiver: http://<server>:<port>/hikvision/event
```
*(Add your screenshot here if available)*

---

## ❓ FINAL QUESTION (VERY IMPORTANT)

**Does your Hikvision device support Event Notification / Alarm Host / HTTP Push?**

---

### 📌 If **YES**:

I will build you:

- ✔ Guaranteed 100% real-time + past sync
- ✔ ERPNext HTTP listener
- ✔ Device pushes events even after 1 week offline
- ✔ No missing logs ever

---

### 📌 If **NO**:

Then ONLY real-time streaming is possible (no past recovery).

But we can still build:

- ✔ Continuous real-time stream
- ✔ Auto reconnect
- ✔ ERPNext stores everything perfectly
- ❌ No past recovery (device doesn’t support it)

---

> **👉 Please reply:**
>
> ✔ **YES** — my device supports Event Push / Alarm Host  
> OR  
> ❌ **NO** — my device does not support Event Push

---

<p align="center">
  <img src="https://img.icons8.com/color/96/000000/api-settings.png" alt="API Icon"/>
  <img src="https://img.icons8.com/color/96/000000/network.png" alt="Network Icon"/>
  <img src="https://img.icons8.com/fluency/96/000000/checked--v1.png" alt="Check Icon"/>
  <img src="https://img.icons8.com/fluency/96/000000/delete-sign.png" alt="Cross Icon"/>
</p>

---

# 🧪 STEP 1 — Identify the Correct API by Auto-Detection

Before you proceed with ERPNext or any integration, **let’s identify which Hikvision API is actually supported by your device!**

Follow these steps **in order** & reply with the result/output from each command (just copy/paste what you see).

---

## 1️⃣ Try the Event API (Real-Time Alert Stream)

```bash
curl --digest -u admin:pnscotton987 "http://172.169.14.19/ISAPI/Event/notification/alertStream"
```

---

## 2️⃣ Try the AccessControl Event API (JSON Event Fetch)

```bash
curl --digest -u admin:pnscotton987 "http://172.169.14.19/ISAPI/AccessControl/Event?format=json"
```

---

## 3️⃣ Try the SDK Event API (If ISAPI not found)

```bash
curl --digest -u admin:pnscotton987 "http://172.169.14.19/SDK/AccessControl/getAllEvent"
```

---

## 4️⃣ Try Time-Attendance Record APIs (JSON Records Fetch)

```bash
curl --digest -u admin:pnscotton987 "http://172.169.14.19/ISAPI/Attendance/Record?format=json"
```

```bash
curl --digest -u admin:pnscotton987 "http://172.169.14.19/ISAPI/AccessControl/AcsEvent?format=json"
```

---

## 🚀 What To Do Now

**Reply with the result of these commands, one-by-one, exactly as your terminal outputs. Example:**

```
1️⃣
curl --digest -u admin:pnscotton987 "http://172.169.14.19/ISAPI/Event/notification/alertStream"
(Output goes here...)

2️⃣
curl --digest -u admin:pnscotton987 "http://172.169.14.19/ISAPI/AccessControl/Event?format=json"
(Output goes here...)
...
```

⚠️ **One of these WILL work** — and then we plug that into your final config!

---

<div align="center">

<img src="https://img.icons8.com/fluency/96/000000/api-settings.png" alt="API Icon" width="48"/>
<img src="https://img.icons8.com/color/96/000000/network.png" alt="Network Icon" width="48"/>
<img src="https://img.icons8.com/ios-filled/100/laboratory.png" alt="Lab Icon" width="48"/>
<img src="https://img.icons8.com/fluency/96/rocket.png" alt="Rocket" width="48"/>

</div>

---



