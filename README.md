# MLflow Setup - Visual Step-by-Step Guide

## 🎯 Goal
Transform your IRIS project from DVC-only to DVC (data) + MLflow (models)

---

## 📊 Before vs After

```
BEFORE:                          AFTER:
┌─────────────┐                  ┌─────────────┐
│    DVC      │                  │    DVC      │
│   ┌─────┐   │                  │   ┌─────┐   │
│   │Data │   │                  │   │Data │   │ ← Still in DVC
│   └─────┘   │                  │   └─────┘   │
│   ┌─────┐   │                  └─────────────┘
│   │Model│   │ ← Remove this    
│   └─────┘   │                  ┌─────────────┐
└─────────────┘                  │   MLflow    │
                                 │   ┌─────┐   │
                                 │   │Model│   │ ← Now in MLflow
                                 │   └─────┘   │
                                 │   ┌─────┐   │
                                 │   │Exps │   │ ← New!
                                 │   └─────┘   │
                                 └─────────────┘
```

---

## 🚀 Super Quick Setup (3 Commands)

```bash
# 1. Download the setup script
chmod +x setup_mlflow_detailed.sh

# 2. Run it
./setup_mlflow_detailed.sh

# 3. Start MLflow UI
./start_mlflow_ui.sh
```

**Done!** Open http://localhost:5000

---

## 📝 Manual Setup (If You Want Control)

### Step 1: Install MLflow (2 minutes)

```bash
# Add to requirements.txt
echo "mlflow==2.9.2" >> requirements.txt
echo "matplotlib==3.8.2" >> requirements.txt
echo "seaborn==0.13.0" >> requirements.txt

# Install
pip install -r requirements.txt

# Verify
python -c "import mlflow; print('✓ MLflow installed!')"
```

**Expected output:**
```
✓ MLflow installed!
```

---

### Step 2: Create MLflow Directory (30 seconds)

```bash
# Create directory for experiments
mkdir -p mlruns

# Update .gitignore
cat >> .gitignore << EOF

# MLflow
mlruns/
mlartifacts/
*.png
EOF
```

**Result:**
```
your-project/
├── mlruns/        ← New! (stores experiments)
└── .gitignore     ← Updated
```

---

### Step 3: Test MLflow (1 minute)

```bash
# Create test file
cat > test_mlflow.py << 'EOF'
import mlflow

mlflow.set_experiment("test")

with mlflow.start_run():
    mlflow.log_param("test", "works")
    mlflow.log_metric("accuracy", 0.95)
    print("✓ MLflow is working!")
EOF

# Run test
python test_mlflow.py

# Clean up
rm test_mlflow.py
```

**Expected output:**
```
✓ MLflow is working!
```

---

### Step 4: Start MLflow UI (10 seconds)

```bash
mlflow ui
```

**Open browser:** http://localhost:5000

**You should see:**
```
┌─────────────────────────────────────┐
│ MLflow                              │
├─────────────────────────────────────┤
│ Experiments                         │
│                                     │
│ ▸ test                    (1 run)  │ ← Your test!
│                                     │
└─────────────────────────────────────┘
```

**If you see this, MLflow is set up! ✅**

---

### Step 5: Remove DVC Model Tracking (30 seconds)

```bash
# Remove model from DVC
git rm iris-dvc-pipeline/model.joblib.dvc
rm iris-dvc-pipeline/model.joblib

# Commit
git commit -m "Remove DVC model tracking, using MLflow now"
```

---

### Step 6: Update Your Code (5 minutes)

Replace these files with the new versions I provided:

1. **src/model_training.py** ← Main changes here
2. **main.py** ← Added MLflow arguments
3. **.github/workflows/ml-pipeline.yml** ← Updated CI/CD

Add these new files:

4. **run_experiments.py** ← New comparison script
5. **tests/test_mlflow_integration.py** ← New tests

---

## ✅ Verification Steps

### Test 1: Basic Training

```bash
python main.py --data-path iris-dvc-pipeline/v1_data.csv
```

**Expected:**
```
INFO - Training model...
INFO - Model logged to MLflow with train accuracy: 0.9667
INFO - Pipeline completed successfully!
INFO - Model accuracy: 0.9500
```

Check MLflow UI → Should see new run in "iris-classification" experiment

---

### Test 2: Hyperparameter Tuning

```bash
python main.py \
  --data-path iris-dvc-pipeline/v1_data.csv \
  --hyperparameter-tuning
```

**Expected:**
```
INFO - Starting hyperparameter tuning...
INFO - Fitting 5 folds for each of X candidates...
INFO - Best params: {'max_depth': 4, 'criterion': 'gini', ...}
INFO - Best CV score: 0.9667
```

Check MLflow UI → Should see nested runs (parent + CV folds)

---

### Test 3: Run Comparisons

```bash
python run_experiments.py --data-path iris-dvc-pipeline/v1_data.csv
```

**Expected:**
```
Running experiment: baseline_shallow
...
Running experiment: deep_tree_exp
...
EXPERIMENT SUMMARY
================================================================================
Experiment Name           Train Acc    Test Acc     Test F1     
--------------------------------------------------------------------------------
hyperparameter_tuning     0.9667       0.9667       0.9661      
deep_tree_exp            0.9778       0.9667       0.9661      
...
✅ All experiments completed!
```

Check MLflow UI → Should see 6 experiments

---

### Test 4: Compare in MLflow UI

1. **Start MLflow UI** (if not running)
   ```bash
   mlflow ui
   ```

2. **Open browser:** http://localhost:5000

3. **Navigate to experiment:**
   - Click "iris-classification-comparison"

4. **Select multiple runs:**
   - Check boxes next to 2+ experiments

5. **Click "Compare" button** at top

6. **You should see:**
   ```
   ┌─────────────────────────────────────────┐
   │ Compare Runs                            │
   ├─────────────────────────────────────────┤
   │                                         │
   │  📊 Parallel Coordinates Plot           │
   │     Shows how parameters affect metrics │
   │                                         │
   │  📈 Scatter Plots                       │
   │     Compare any two metrics             │
   │                                         │
   │  📋 Table View                          │
   │     All params and metrics side-by-side │
   │                                         │
   └─────────────────────────────────────────┘
   ```

---

## 🎓 Your First Real Workflow

```bash
# Terminal 1: Start MLflow UI
mlflow ui

# Terminal 2: Run experiments
python run_experiments.py --data-path iris-dvc-pipeline/v1_data.csv

# Browser: View results
# Open http://localhost:5000
# Select multiple runs → Compare
```

---

## 🎨 MLflow UI Quick Tour

### Experiments Page
```
┌──────────────────────────────────────────────────┐
│ MLflow                                           │
├──────────────────────────────────────────────────┤
│ Experiments           Search: [____________]     │
│                                                  │
│ ▾ iris-classification                (12 runs)  │ ← Your experiments
│   • run_001  acc: 0.95  depth: 3                │
│   • run_002  acc: 0.96  depth: 4                │
│   • run_003  acc: 0.94  depth: 2                │
│                                                  │
│ ▾ iris-comparison                     (6 runs)  │
│   ...                                            │
└──────────────────────────────────────────────────┘
```

### Run Details Page
```
┌──────────────────────────────────────────────────┐
│ Run: run_001                                     │
├──────────────────────────────────────────────────┤
│ Parameters          │ Metrics                    │
│ • max_depth: 3      │ • test_accuracy: 0.9500   │
│ • criterion: gini   │ • test_f1_score: 0.9495   │
│ • random_state: 42  │ • train_accuracy: 0.9667  │
│                                                  │
│ Artifacts                                        │
│ • model/                                         │
│ • confusion_matrix.png                           │
└──────────────────────────────────────────────────┘
```

### Compare Page
```
┌──────────────────────────────────────────────────┐
│ Comparing 4 runs                                 │
├──────────────────────────────────────────────────┤
│                                                  │
│        Parallel Coordinates Plot                 │
│                                                  │
│  max_depth ────────────── test_accuracy          │
│      │                           │               │
│      2 ─────────────────────── 0.94              │
│      │    ╱                                      │
│      3 ───────────────────── 0.95                │
│      │      ╲                                    │
│      4 ─────────────────────── 0.96              │
│      │            ╲                              │
│      5 ─────────────────────── 0.95              │
│                                                  │
│ 💡 Shows: max_depth=4 gives best accuracy!       │
└──────────────────────────────────────────────────┘
```

---

## 🔧 Troubleshooting Visual Guide

### Issue: Can't see experiments in UI

```
Problem:                          Solution:
┌──────────────┐                  ┌──────────────┐
│ UI is empty  │                  │ Check:       │
│ No experiments│    ────────────>│ 1. Tracking  │
│              │                  │    URI       │
└──────────────┘                  │ 2. mlruns/   │
                                  │    directory │
                                  └──────────────┘

Command:
python -c "import mlflow; print(mlflow.get_tracking_uri())"

Expected: file:///path/to/mlruns
```

### Issue: Import error

```
Error:                            Solution:
┌──────────────┐                  ┌──────────────┐
│ ModuleNotFound│                 │ pip install  │
│ No module     │   ─────────────>│ -r           │
│ named mlflow  │                 │ requirements │
└──────────────┘                  │ .txt         │
                                  └──────────────┘
```

### Issue: Can't start MLflow UI

```
Error:                            Solution:
┌──────────────┐                  ┌──────────────┐
│ Port 5000     │                 │ Use different│
│ already in    │   ─────────────>│ port:        │
│ use           │                 │ mlflow ui    │
└──────────────┘                  │ --port 5001  │
                                  └──────────────┘
```

---

## 📊 Progress Checklist

Use this to track your setup:

```
Setup Progress:
├─ [✓] Python 3.8+ installed
├─ [✓] pip working
├─ [✓] MLflow installed
├─ [✓] mlruns directory created
├─ [✓] .gitignore updated
├─ [✓] Test experiment created
├─ [✓] MLflow UI accessible
├─ [✓] DVC model tracking removed
├─ [✓] Code files updated
├─ [✓] Can train model
├─ [✓] Can run comparisons
├─ [✓] Can view in UI
└─ [✓] Can load from registry

✅ All done!
```

---

## 🎯 What Success Looks Like

After setup, you should be able to:

1. **Train a model:**
   ```bash
   python main.py --data-path iris-dvc-pipeline/v1_data.csv
   ```
   → See run in MLflow UI ✅

2. **Run hyperparameter tuning:**
   ```bash
   python main.py --hyperparameter-tuning
   ```
   → See CV results in MLflow UI ✅

3. **Compare experiments:**
   ```bash
   python run_experiments.py
   ```
   → See 6 experiments, can compare in UI ✅

4. **Load model from registry:**
   ```bash
   python main.py --use-mlflow-model
   ```
   → Uses Production model ✅

---

## 🚀 Next Steps After Setup

```
1. Run Experiments
   │
   ├─> python run_experiments.py
   │
2. View in MLflow UI
   │
   ├─> mlflow ui → http://localhost:5000
   │
3. Compare Results
   │
   ├─> Select runs → Click Compare
   │
4. Find Best Model
   │
   ├─> Sort by test_accuracy
   │
5. Promote to Production
   │
   └─> Use promote_model_to_production()
```

---

## 💡 Pro Tips

1. **Keep MLflow UI running** in a dedicated terminal
2. **Use helper scripts** for common tasks
3. **Check UI after each experiment** to see results
4. **Compare runs frequently** to understand what works
5. **Tag important runs** for easy finding later

---

## 📚 Where to Go Next

- **Full documentation:** `MLFLOW_INTEGRATION.md`
- **Migration guide:** `MIGRATION_GUIDE.md`
- **Quick commands:** `QUICK_REFERENCE.md`
- **MLflow docs:** https://mlflow.org/docs

---

## 🎉 You're Ready!

If you can:
- ✅ Start MLflow UI
- ✅ See experiments
- ✅ Train models
- ✅ Compare runs

**Then you're all set!** Start experimenting! 🚀