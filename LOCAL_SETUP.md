# Local Setup Guide - RAG Company Policies

Complete step-by-step guide to set up and run the RAG system on your local machine.

---

## 📋 Prerequisites

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **OS** | Windows 10, macOS 10.15, Ubuntu 20.04 | Windows 11, macOS 13+, Ubuntu 22.04 |
| **Python** | 3.11.0 | 3.11.x (latest) |
| **RAM** | 8 GB | 16 GB |
| **Disk Space** | 10 GB free | 20 GB free |
| **Internet** | Required for AWS Bedrock | Stable connection |

### Required Accounts

1. **AWS Account** with:
   - Bedrock access enabled
   - Programmatic access (Access Key + Secret)
   - Budget: ~$5/month for testing

2. **GitHub Account** (for version control)

---

## 🔧 Step 1: Install Python 3.11

### Windows

1. **Download Python 3.11:**
   - Go to: https://www.python.org/downloads/
   - Download Python 3.11.x (latest)

2. **Install:**
   - Run installer
   - ✅ Check "Add Python to PATH"
   - Click "Install Now"

3. **Verify:**
```bash
