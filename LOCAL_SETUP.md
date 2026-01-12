# Local Setup Guide - RAG Company Policies

Complete step-by-step guide to set up and run the RAG system on your local machine.

---

##  Prerequisites

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
   - Budget: ~€5/month for testing

2. **GitHub Account** (for version control)

---

##  Step 1: Install Python 3.11

### Windows

1. **Download Python 3.11:**
   - Go to: https://www.python.org/downloads/
   - Download Python 3.11.x (latest)

2. **Install:**
   - Run installer
   -  Check "Add Python to PATH"
   - Click "Install Now"

3. **Verify:**
```bash
python --version
# Should show: Python 3.11.x
```

---

##  Step 2: Clone the Repository

### Option A: Using Git (Recommended)
```bash
# Install Git if needed
# Windows: https://git-scm.com/download/win
# Mac: brew install git
# Linux: sudo apt install git

# Clone repository
git clone https://github.com/YOUR_USERNAME/rag-company-policies.git
cd rag-company-policies
```

### Option B: Download ZIP

From GitHub
---

##  Step 3: Create Virtual Environment

### Why Virtual Environment?
- Isolates project dependencies
- Prevents version conflicts
- Easy to recreate

### Create and Activate

**Windows:**
```bash
# Create virtual environment
python -m venv venv

# Activate
venv\Scripts\activate

# You should see (venv) in your prompt
```
### Verify Activation
```bash
which python
# Should show path inside venv folder
```

---

##  Step 4: Install Dependencies

### Core Dependencies
```bash
# Upgrade pip first
pip install --upgrade pip

# Install all requirements
pip install -r requirements.txt
```

**This will install:**
- LangChain framework
- FAISS vector database
- PDF processing libraries
- Sentence transformers
- AWS SDK (boto3)

### Verify Installation

### Common Installation Issues

#### Issue 1: FAISS installation fails on Windows

**Error:**
```
ERROR: Could not build wheels for faiss-cpu
```

**Solution:**
```bash
# Install Visual C++ Build Tools
# Download from: https://visualstudio.microsoft.com/visual-cpp-build-tools/

# Or use conda
conda install -c conda-forge faiss-cpu
```

#### Issue 2: PyMuPDF installation fails

**Error:**
```
ERROR: Failed building wheel for pymupdf
```

**Solution:**
```bash
pip install --upgrade pip setuptools wheel
pip install pymupdf --no-cache-dir
```

---

##  Step 5: Configure AWS Credentials

### Create AWS Access Keys

1. **Go to AWS Console:**
   - Navigate to IAM → Users → Your User
   - Click "Security credentials"
   - Click "Create access key"
   - Select "Application running outside AWS"
   - Save Access Key ID and Secret Access Key

### Configure AWS CLI

**Option A: Using AWS CLI (Recommended)**
```bash
# Install AWS CLI
# Windows: https://aws.amazon.com/cli/
# Mac: brew install awscli
# Linux: sudo apt install awscli

# Configure credentials
aws configure

# Enter when prompted:
AWS Access Key ID: YOUR_ACCESS_KEY
AWS Secret Access Key: YOUR_SECRET_KEY
Default region name: us-east-1
Default output format: json
```

**Option B: Manual Configuration**

Create `~/.aws/credentials` file:

**Windows:** `C:\Users\YOUR_USERNAME\.aws\credentials`
**Mac/Linux:** `~/.aws/credentials`
```ini
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
region = us-east-1
```

### Verify AWS Configuration
```bash
aws sts get-caller-identity
```

**Expected output:**
```json
{
    "UserId": "AIDAXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/your-name"
}
```

---

##  Step 6: AWS Bedrock Models

### Verify Model Access
```bash
# Test Titan Embeddings
aws bedrock list-foundation-models --region us-east-1 | grep -i titan

# Test Llama 3
aws bedrock list-foundation-models --region us-east-1 | grep -i llama
```

### Test Bedrock Connection

Create `test_bedrock.py`:
```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

# Test Llama 3
response = bedrock.invoke_model(
    modelId='meta.llama3-70b-instruct-v1:0',
    body=json.dumps({
        "prompt": "<|begin_of_text|><|start_header_id|>user<|end_header_id|>\n\nHello!<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n",
        "max_gen_len": 50,
        "temperature": 0.1
    })
)

result = json.loads(response['body'].read())
print(" Bedrock connection successful!")
print(f"Response: {result['generation']}")
```

Run test:
```bash
python test_bedrock.py
```

---

##  Step 7: Prepare Sample Documents

### Create Test Documents Directory
```bash
mkdir test_documents
cd test_documents
```

### Document Naming Convention

**Format:** `DEPT-TYPE-NUMBER_Description.pdf`

**Examples:**
- `FIN-POL-001_TravelExpensePolicy.pdf`
- `LEG-POL-001_DataPrivacyPolicy.pdf`
- `HR-POL-001_VacationPolicy.pdf`
- `IT-POL-001_PasswordPolicy.pdf`

### Department Codes

| Code | Department | Code | Department |
|------|-----------|------|-----------|
| FIN | Finance | HR | Human Resources |
| LEG | Legal | IT | IT |
| MKT | Marketing | SA | Sales |
| OPS | Operations | PROC | Procurement |
| QM | Quality | PD | Production |

### Sample Document Content

Create a test PDF with this content (using Word/Google Docs):

**File:** `FIN-POL-001_TravelExpensePolicy.pdf`
```
TECHCORP GMBH
Reisekostenrichtlinie

1. FAHRTKOSTEN

1.1 Privat-PKW
Für die Nutzung von privaten Fahrzeugen wird eine Kilometerpauschale von 0,30 EUR pro Kilometer erstattet.

1.2 Bahnfahrten
Bahnfahrten in der 2. Klasse werden vollständig erstattet. Bei Reisen über 3 Stunden ist die 1. Klasse zulässig.

2. ÜBERNACHTUNGEN

Hotelübernachtungen werden bis zu einem Höchstbetrag von 120 EUR pro Nacht erstattet.

3. VERPFLEGUNG

Verpflegungspauschalen:
- Über 8 Stunden Abwesenheit: 14 EUR
- Über 12 Stunden Abwesenheit: 28 EUR
- Über 24 Stunden Abwesenheit: 42 EUR
```

**Minimum:** 3-5 PDF files for testing

---

##  Step 8: Build FAISS Index

### Run Document Processing
```bash
# Make sure you're in project root
cd rag-company-policies

# Run processing script
python src/process_documents.py
```

**Expected output:**
```
============================================================
RAG Data Ingestion with Metadata - TechCorp Policies
============================================================

[1/5] Loading PDF documents from test_documents/...
✓ Loaded 15 pages from PDFs

[2/5] Adding department and document type metadata...

 Metadata Summary:
  - Finance: 5 pages
  - Legal: 4 pages
  - HR: 6 pages

[3/5] Splitting documents into chunks...
✓ Created 45 chunks

[4/5] Initializing Bedrock Titan Embeddings...
✓ Embeddings model ready

[5/5] Building FAISS vector index with metadata...
 Processing 45 chunks...
✓ FAISS index created successfully
✓ Index saved to: faiss_indexes/company_policies

============================================================
Testing Metadata Filtering
============================================================

 Query: 'Reisekostenrichtlinie' (No filter)
Results:
  1. Finance - FIN-POL-001_TravelExpensePolicy.pdf
  2. Finance - FIN-POL-001_TravelExpensePolicy.pdf
  3. Finance - FIN-POL-001_TravelExpensePolicy.pdf

============================================================
 Pipeline completed successfully!
============================================================
```

### Verify FAISS Index Creation
```bash
# Check if index files were created
ls faiss_indexes/company_policies/

# Should show:
# index.faiss
# index.pkl
```

**File sizes:**
- `index.faiss`: ~1-10 MB (depending on document count)
- `index.pkl`: ~100 KB - 2 MB

---

##  Step 9: Test Query System

### Run Interactive Query Interface
```bash
python src/query_rag.py
```

**Expected output:**
```
======================================================================
 RAG Query Interface - Enhanced Formatting
======================================================================

 Loading FAISS index...
 FAISS index loaded
 Initializing Llama...
 Llama ready
 Loading reranker model...
 Reranker loaded

model: llama

======================================================================
 Enter your question (or 'exit'):
>
```

### Test Queries

**Test 1: German Query**
```
> Wie hoch ist die Kilometerpauschale für Privatfahrzeuge?

 German detected
 Department: Finance
 Retrieving from FAISS...
 Found 8 candidates

 Reranking 8 candidates...

 Reranking Results:
======================================================================
Rank #1 ↑ (was #2)
  Rerank: 8.4523 | Original: 120.45
  FIN-POL-001_TravelExpensePolicy.pdf, Page 1

Rank #2 → (was #2)
  Rerank: 7.8912 | Original: 125.32
  FIN-POL-001_TravelExpensePolicy.pdf, Page 1

 Generating answer...

======================================================================
 ANTWORT
======================================================================
Die Reisekostenrichtlinie regelt die Erstattung von Geschäftsreisen.

FAHRTKOSTEN
-----------
- Privat-PKW: 0,30 EUR pro Kilometer
- Bahnfahrten in 2. Klasse werden vollständig erstattet
- Bei Reisen über 3 Stunden ist 1. Klasse zulässig

======================================================================
 QUALITÄT
======================================================================
 Vertrauensgrad: Hoch
📄 Quellen: 4
  • FIN-POL-001_TravelExpensePolicy.pdf, Seite 1 (Score: 120.5)
  • FIN-POL-001_TravelExpensePolicy.pdf, Seite 2 (Score: 125.3)
```

**Test 2: English Query**
```
> What is the mileage reimbursement rate?

 English detected - translating for retrieval...
 Retrieval query: Wie hoch ist die Kilometerpauschale?
 Department: Finance
...

 ANSWER
The travel expense policy regulates reimbursement for business trips.

TRAVEL COSTS
------------
- Private vehicle: €0.30 per kilometer
- Train travel in 2nd class is fully reimbursed
- 1st class is permitted for journeys over 3 hours
```

### Exit Query Interface
```
> exit
 Goodbye!
```

---

##  Step 10: Run Tests

### Create Test File

Create `tests/test_queries.py`:
```python
import sys
sys.path.append('src')

from query_rag import load_vectorstore, detect_language, detect_department

def test_faiss_loading():
    """Test FAISS index loads correctly"""
    try:
        vs = load_vectorstore()
        assert vs is not None
        print(" FAISS loading test passed")
    except Exception as e:
        print(f" FAISS loading test failed: {e}")

def test_language_detection():
    """Test language detection"""
    assert detect_language("Wie hoch ist die Kilometerpauschale?") == "de"
    assert detect_language("What is the mileage rate?") == "en"
    print(" Language detection test passed")

def test_department_detection():
    """Test department detection"""
    assert detect_department("Reisekostenrichtlinie") == "Finance"
    assert detect_department("Datenschutz") == "Legal"
    assert detect_department("Urlaubstage") == "HR"
    print(" Department detection test passed")

def test_search():
    """Test vector search"""
    vs = load_vectorstore()
    results = vs.similarity_search("Kilometerpauschale", k=3)
    assert len(results) > 0
    print(f" Search test passed - found {len(results)} results")

if __name__ == "__main__":
    print("Running tests...\n")
    test_faiss_loading()
    test_language_detection()
    test_department_detection()
    test_search()
    print("\n All tests passed!")
```

### Run Tests
```bash
python tests/test_queries.py
```

**Expected output:**
```
Running tests...

 FAISS loading test passed
 Language detection test passed
 Department detection test passed
 Search test passed - found 3 results

 All tests passed!
```

---

##  Step 11: Optional - Jupyter Notebook Demo

### Install Jupyter
```bash
pip install jupyter notebook ipywidgets
```

### Create Demo Notebook

Create `notebooks/demo.ipynb`:
```python
# Cell 1: Setup
import sys
sys.path.append('../src')

from query_rag import load_vectorstore, load_llm, load_reranker
import pandas as pd

print("Loading models...")
vs = load_vectorstore()
llm = load_llm("llama")
reranker = load_reranker()
print(" Ready!")

# Cell 2: Interactive Query
question = "Wie hoch ist die Kilometerpauschale?"

# Search
results = vs.similarity_search_with_score(question, k=5)

# Display results
df = pd.DataFrame([
    {
        'Filename': doc.metadata['filename'],
        'Page': doc.metadata['page'],
        'Department': doc.metadata['department'],
        'Score': score,
        'Preview': doc.page_content[:100] + '...'
    }
    for doc, score in results
])

display(df)

# Cell 3: Visualize Scores
import matplotlib.pyplot as plt

scores = [score for _, score in results]
filenames = [doc.metadata['filename'] for doc, _ in results]

plt.figure(figsize=(10, 5))
plt.barh(range(len(scores)), scores)
plt.yticks(range(len(scores)), filenames)
plt.xlabel('Similarity Score (lower is better)')
plt.title('Document Relevance Scores')
plt.gca().invert_yaxis()
plt.tight_layout()
plt.show()
```

### Run Notebook
```bash
jupyter notebook notebooks/demo.ipynb
```

---

##  Troubleshooting

### Issue 1: "ModuleNotFoundError: No module named 'faiss'"

**Solution:**
```bash
pip install faiss-cpu --force-reinstall
```

### Issue 2: "botocore.exceptions.NoCredentialsError"

**Solution:**
```bash
# Check credentials file
cat ~/.aws/credentials  # Linux/Mac
type %USERPROFILE%\.aws\credentials  # Windows

# Re-configure
aws configure
```

### Issue 3: "ResourceNotFoundException: Could not find model"

**Solution:**
- Verify region is `us-east-1`
- Check model access in Bedrock console
- Wait 5-10 minutes for approval

### Issue 4: FAISS index not found

**Solution:**
```bash
# Re-run document processing
python src/process_documents.py

# Verify files exist
ls faiss_indexes/company_policies/
```

### Issue 5: Out of memory error

**Solution:**
```python
# In process_documents.py, reduce batch size:
# Change from:
embeddings = embed_documents(all_chunks)

# To:
batch_size = 10
for i in range(0, len(chunks), batch_size):
    batch = chunks[i:i+batch_size]
    embeddings = embed_documents(batch)
```

---

##  Verification Checklist

Before moving to deployment, verify:

- [ ] Python 3.11 installed
- [ ] Virtual environment activated
- [ ] All dependencies installed
- [ ] AWS credentials configured
- [ ] Bedrock models enabled
- [ ] Test documents prepared (3+ PDFs)
- [ ] FAISS index created successfully
- [ ] Query system works (tested 3+ queries)
- [ ] Tests pass
- [ ] Both German and English queries work

---

##  Next Steps

Once local setup is complete:

1. **Push to GitHub:**
```bash
git add .
git commit -m "Initial commit - RAG system"
git push origin main
```

2. **Deploy to AWS Lambda:**
   - See `AWS_DEPLOYMENT.md`

3. **Share your project:**
   - Add to LinkedIn
   - Add to portfolio website
   - Share on Twitter/X

---

##  Tips for Success

### Development Workflow
```bash
# Always activate venv first
source venv/bin/activate  # Mac/Linux
venv\Scripts\activate     # Windows

# Run your code
python src/query_rag.py

# Deactivate when done
deactivate
```

### Keep Dependencies Updated
```bash
# Check outdated packages
pip list --outdated

# Update all
pip install --upgrade -r requirements.txt

# Save updated versions
pip freeze > requirements.txt
```

### Git Best Practices
```bash
# Create .gitignore first
echo "venv/" >> .gitignore
echo "*.pyc" >> .gitignore
echo "faiss_indexes/" >> .gitignore
echo ".env" >> .gitignore

# Commit frequently
git add .
git commit -m "Descriptive message"
git push
```

---

##  Learning Resources

### FAISS
- [FAISS Documentation](https://faiss.ai/)
- [FAISS Tutorial](https://www.pinecone.io/learn/faiss/)

### LangChain
- [LangChain Docs](https://python.langchain.com/)
- [RAG Tutorial](https://python.langchain.com/docs/tutorials/rag/)

### AWS Bedrock
- [Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)

---

**Setup complete!**

Your local RAG system is now ready for development and testing.

Need help? Check the troubleshooting section or open an issue on GitHub.
