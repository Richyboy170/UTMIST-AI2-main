# Quick Start Guide - UTMIST AI² Tournament

Get up and running with the AI² tournament in 4 simple steps!

## 1. Install Requirements

Install all required dependencies using pip:

```bash
pip install -r requirements.txt
```

**What this installs:**
- PyTorch (deep learning framework)
- Stable-Baselines3 (reinforcement learning library)
- Pygame & PyMunk (game environment)
- And other necessary dependencies

**Troubleshooting:**
- If you encounter permission errors, try: `pip install -r requirements.txt --user`
- For Python version issues, ensure you're using Python 3.8 or higher
- Consider using a virtual environment to avoid conflicts

---

## 2. Train Your Model

Run the training script to start training your RL agent:

```bash
python user/train_agent.py
```

**What happens during training:**
- Your agent plays against various opponents (70% self-play, 20% rule-based, 10% random)
- Models are automatically saved every 50,000 steps
- Saved checkpoints appear in `checkpoints/experiment_optimized/`
- Training visualizations and logs are generated

**Training from scratch vs. continuing:**

**Start from scratch** (default):
```python
# In train_agent.py (line 714)
my_agent = CustomAgent(sb3_class=PPO, extractor=MLPExtractor)
```

**Continue from a checkpoint:**
```python
# In train_agent.py (line 714)
my_agent = CustomAgent(
    sb3_class=PPO,
    extractor=MLPExtractor,
    file_path='checkpoints/experiment_optimized/rl_model_500000_steps.zip'
)
```

**Key configuration options:**
- `train_timesteps`: Total training steps (default: 1 billion)
- `save_freq`: How often to save (default: every 50k steps)
- `opponent_cfg`: Distribution of opponent types
- `reward_manager`: Customize reward functions in `gen_reward_manager()`

**Training tips:**
- Training can take several hours to days depending on your hardware
- Use Ctrl+C to stop training (your progress will be saved)
- Monitor the console for win rates and reward metrics
- GPU highly recommended for faster training

---

## 3. Test Your Model

### Option A: Watch Your Agent Battle (Demo Mode)

Test your trained agent against different opponents with visual feedback:

```bash
python user/demo_match.py
```

**Before running, edit `demo_match.py`:**
```python
# Line 11: Load your trained model
opponent = SubmittedAgent(file_path='checkpoints/experiment_optimized/rl_model_500000_steps.zip')

# Line 8: Choose who you want to control
my_agent = UserInputAgent()  # You play with keyboard controls
# OR
my_agent = SubmittedAgent(file_path='path/to/another/model.zip')  # Two AI agents battle
```

**Controls (if using UserInputAgent):**
- **WASD**: Move (W=up, A=left, S=down, D=right)
- **Space**: Jump
- **J**: Attack
- **K**: Special move
- **L**: Another action
- **G**: Pick up/drop weapon

### Option B: Real-time PvP Match

Play against your agent in real-time:

```bash
python user/pvp_match.py
```

Edit `pvp_match.py` to configure:
```python
# Line 13: Your agent (keyboard controlled)
my_agent = UserInputAgent()

# Line 14: Opponent agent (your trained model)
opponent = SubmittedAgent(file_path='checkpoints/experiment_optimized/rl_model_500000_steps.zip')
```

### Option C: Run Automated Battles

Run headless battles for testing/evaluation:

```bash
python user/battle.py
```

This runs battle tests between two agents and updates ELO ratings.

### Option D: Run Validation Tests

Ensure your agent meets submission requirements:

```bash
pytest user/validate.py
```

This validates:
- Your agent loads correctly
- It can make predictions
- No import errors or crashes occur

---

## 4. Deploy/Submit Your Model

### Option 1: Upload to Google Drive (Recommended for Tournament)

1. **Find your best model checkpoint:**
   ```bash
   ls -lh checkpoints/experiment_optimized/
   ```
   Look for files like `rl_model_500000_steps.zip`

2. **Upload to Google Drive:**
   - Go to [drive.google.com](https://drive.google.com)
   - Upload your `.zip` model file
   - Right-click the file → "Get link" → "Anyone with the link"
   - Copy the share link

3. **Update your submission file (`user/my_agent.py`):**
   ```python
   def _gdown(self) -> str:
       data_path = "rl-model.zip"
       if not os.path.isfile(data_path):
           print(f"Downloading {data_path}...")
           # Replace with YOUR Google Drive link
           url = "https://drive.google.com/file/d/YOUR_FILE_ID_HERE/view?usp=sharing"
           gdown.download(url, output=data_path, fuzzy=True)
       return data_path
   ```

4. **Set your default model path:**
   ```python
   # In user/my_agent.py, line 36
   def _initialize(self) -> None:
       if self.file_path is None:
           # Option 1: Download from Google Drive
           self.file_path = self._gdown()
           self.model = PPO.load(self.file_path)

           # Option 2: Use local path
           # self.model = PPO.load('checkpoints/experiment_optimized/rl_model_500000_steps.zip')
       else:
           self.model = PPO.load(self.file_path)
   ```

### Option 2: Include Model Locally

If submitting code directly (not via download):

1. Copy your best model to a known location:
   ```bash
   cp checkpoints/experiment_optimized/rl_model_500000_steps.zip models/my_best_agent.zip
   ```

2. Update `user/my_agent.py`:
   ```python
   def __init__(self, file_path: Optional[str] = None):
       # Set default path to your model
       if file_path is None:
           file_path = 'models/my_best_agent.zip'
       super().__init__(file_path)
   ```

### Option 3: Share via GitHub/GitLab

1. **Commit your trained model:**
   ```bash
   git add checkpoints/experiment_optimized/rl_model_500000_steps.zip
   git commit -m "Add trained model checkpoint"
   git push
   ```

2. **Note:** Large model files (>100MB) may require Git LFS:
   ```bash
   git lfs install
   git lfs track "*.zip"
   git add .gitattributes
   git commit -m "Track model files with Git LFS"
   ```

### Submission Checklist

Before submitting, verify:
- [ ] Your `SubmittedAgent` class in `user/my_agent.py` is properly configured
- [ ] Model loads without errors: `python -c "from user.my_agent import SubmittedAgent; agent = SubmittedAgent()"`
- [ ] Validation tests pass: `pytest user/validate.py`
- [ ] Google Drive link is public and accessible
- [ ] You've tested your agent in a demo match

---

## Quick Reference

| Task | Command |
|------|---------|
| Install dependencies | `pip install -r requirements.txt` |
| Train from scratch | `python user/train_agent.py` |
| Test with demo | `python user/demo_match.py` |
| Play vs your agent | `python user/pvp_match.py` |
| Run validation | `pytest user/validate.py` |
| Find saved models | `ls checkpoints/experiment_optimized/` |

## Need Help?

- **Discord**: Ask questions in the UTMIST Discord server
- **Documentation**: Check `guides/technical_guide.md` for detailed info
- **Colab Notebook**: [Interactive tutorial](https://colab.research.google.com/drive/1V184vtHSagN13L0SbWGmnY-jCDvIefmm?usp=sharing)
- **Paper**: [Full documentation](https://drive.google.com/file/d/1G0hatGPBXvh2j5byjfrqKBthknNOt5sp/view)

Good luck with your agent! 🎮🤖
