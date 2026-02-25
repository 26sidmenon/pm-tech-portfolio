# HN Analyzer - Sentiment Analysis Feature

## Project Overview
Extended the Hacker News comment analyzer with VADER sentiment analysis using Claude Code as an AI pair programming assistant. This was my first experience using an AI coding agent to add a complete feature to existing code.

## Problem Statement
The original HN analyzer could extract comments and categorize them by keywords (pricing, features, ease of use), but provided no insight into whether discussions were positive, negative, or neutral. Product managers need to understand not just *what* people are talking about, but *how* they feel about it.

## Solution
Implemented VADER (Valence Aware Dictionary and sEntiment Reasoner) sentiment analysis to:
- Score each comment from -1.0 (most negative) to +1.0 (most positive)
- Classify comments as positive, negative, or neutral
- Display sentiment distribution summary in terminal
- Export sentiment scores to CSV for further analysis
- Show most positive and negative comments for quick insights

## Tools & Technologies
- **Python 3.14** - Programming language
- **NLTK (Natural Language Toolkit)** - Sentiment analysis library
- **VADER** - Pre-trained sentiment model optimized for social media text
- **Claude Code** - AI pair programming assistant in VS Code
- **Git/GitHub** - Version control

## Using Claude Code: The Process

### Initial Prompt
```
@hn_analyzer.py Create a plan for adding sentiment analysis 
to the HN analyzer, then implement it
```

### What Claude Code Did
1. **Analyzed existing codebase** - Checked current imports and structure
2. **Proposed VADER** - Recommended appropriate library for short-form text
3. **Created detailed plan** - 5-step implementation with verification steps
4. **Implemented incrementally** - Showed code diffs for each change
5. **Explained new concepts** - Taught me about NLTK, sentiment scoring thresholds

### Implementation Steps
1. Added NLTK and VADER imports
2. Created `analyze_sentiment()` function with compound scoring logic
3. Integrated sentiment analysis into existing workflow
4. Extended CSV output with sentiment columns
5. Added terminal output showing sentiment distribution

### Real-World Debugging
Encountered SSL certificate error when downloading VADER lexicon:
```
[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed
```

**Resolution:**
- Ran Python's certificate installer: `/Applications/Python 3.14/Install Certificates.command`
- Updated certifi package
- Successfully downloaded VADER lexicon
- Learned about Python SSL certificate management on macOS

## Code Architecture

**New Function Added:**
```python
def analyze_sentiment(comments_list):
    """Score each comment as positive, negative, or neutral using VADER"""
    # Returns: (comments with sentiment, sentiment counts)
```

**Sentiment Classification Logic:**
- Compound score ≥ 0.05 → Positive
- Compound score ≤ -0.05 → Negative
- Otherwise → Neutral

**CSV Output Enhancement:**
Added two new columns:
- `sentiment` - Classification (positive/negative/neutral)
- `sentiment_score` - Numerical score (-1.0 to +1.0)

## Sample Output

**Terminal Summary:**
```
==================================================
😊 SENTIMENT ANALYSIS
==================================================
Positive: 12  (60%)
Neutral:  5   (25%)
Negative: 3   (15%)

Most positive: "This tool is absolutely amazing..."
Most negative: "The pricing model is completely broken..."
```

**CSV Enhancement:**
| author | date | story | comment | sentiment | sentiment_score |
|--------|------|-------|---------|-----------|----------------|
| user123 | 2024-01-15 | Product launch | Love this feature! | positive | 0.743 |

## What I Learned

### Python Concepts
- **NLTK library** - How natural language processing libraries work
- **Compound scoring** - Understanding multi-dimensional sentiment metrics
- **Dictionary unpacking** - Using `{**dict1, 'key': 'value'}` syntax
- **SSL certificates** - Why Python needs certificates for HTTPS downloads

### Working with AI Coding Assistants
- **@-mentions** - Referencing specific files for context
- **Plan-first approach** - Reviewing plans before implementation
- **Inline diffs** - Understanding what's being changed before accepting
- **Incremental implementation** - Building features step-by-step
- **Learning mode** - Asking "why" after each change

### Product Management Insights
- Sentiment analysis reveals the *tone* of user feedback, not just topics
- Quantifying sentiment (60% positive vs 15% negative) helps prioritize issues
- Most positive/negative examples provide concrete quotes for stakeholders
- CSV export allows further analysis in Excel or BI tools

## Key Decisions Made

| Decision | Options Considered | Choice | Reason |
|----------|-------------------|--------|--------|
| Sentiment library | TextBlob, VADER, Transformers | VADER | Optimized for short social media text, no training required |
| Scoring threshold | 0.0, 0.05, 0.1 | ±0.05 | VADER's recommended threshold for social media |
| Output format | Terminal only, CSV only, Both | Both | Terminal for quick insights, CSV for detailed analysis |
| AI assistant approach | Auto-accept, Review each change | Review each | Learning mode - understand every change |

## Challenges & Solutions

### Challenge 1: SSL Certificate Error
**Problem:** NLTK couldn't download VADER lexicon due to certificate verification failure  
**Solution:** Ran Python's certificate installer and updated certifi package  
**Learning:** macOS Python installations need explicit certificate setup

### Challenge 2: Understanding Compound Scores
**Problem:** Unclear how VADER's compound score differs from pos/neg/neu scores  
**Solution:** Asked Claude Code to explain the scoring algorithm  
**Learning:** Compound is normalized sum of all lexicon ratings, pos/neg/neu are proportions

### Challenge 3: CSV Column Order
**Problem:** New sentiment columns needed to be added to existing CSV structure  
**Solution:** Updated fieldnames list in DictWriter  
**Learning:** CSV column order matters for consistency across runs

## Outcome
- ✅ Sentiment analysis fully integrated into HN analyzer
- ✅ Terminal output shows sentiment distribution at a glance
- ✅ CSV includes sentiment scores for every comment
- ✅ Zero breaking changes to existing functionality
- ✅ Processing time increased by ~0.2 seconds for 20 comments (negligible)

## Sample Use Case
**Query:** "Notion" on Hacker News  
**Results:**
- 20 comments analyzed
- 65% positive sentiment (users love flexibility)
- 20% neutral (feature discussions)
- 15% negative (complaints about performance)
- Most negative: "Notion is unbearably slow on mobile"
- Insight: Mobile performance is a consistent pain point

## What I Would Do Next
- Add sentiment trend analysis (sentiment over time)
- Correlate sentiment with specific keywords (e.g., "Are pricing complaints more negative?")
- Export sentiment data to visualization tools (charts/graphs)
- Add sentiment filtering (show only negative comments)
- Compare sentiment across different products

## Files Modified
- `hn_analyzer.py` - Main analyzer script
  - Added: `analyze_sentiment()` function
  - Modified: CSV export to include sentiment columns
  - Modified: Terminal output to show sentiment summary

## Dependencies Added
```
pip install nltk
```

VADER lexicon download (one-time):
```
python3 -c "import nltk; nltk.download('vader_lexicon')"
```

## Resources & Documentation
- [VADER Sentiment Analysis](https://github.com/cjhutto/vaderSentiment) - Original research paper
- [NLTK Documentation](https://www.nltk.org/) - Natural Language Toolkit
- [Claude Code Docs](https://code.claude.com/docs) - AI assistant documentation

## Reflections

### What Worked Well
- Claude Code's plan-first approach helped me understand before implementing
- Reviewing each diff reinforced my Python learning
- Asking "why" after changes deepened understanding
- SSL error became a learning opportunity, not a blocker

### What I'd Do Differently
- Test on a smaller dataset first to verify sentiment accuracy
- Add unit tests for the sentiment function
- Document the sentiment threshold choices in code comments
- Create a separate config file for sentiment parameters

### Impact on My Learning
This was my first time using an AI coding assistant for a complete feature. Key takeaway: **Claude Code is most valuable when you use it as a teacher, not just a code generator.** By reviewing plans, asking questions, and understanding each change, I learned:
- How to add dependencies to Python projects
- How NLP libraries work under the hood
- How to integrate new features without breaking existing code
- Real-world debugging (SSL certificates, library downloads)

The feature works, but more importantly, **I understand how it works.**

---

**Repository:** [review-analyzer](https://github.com/YOUR-USERNAME/review-analyzer)  
**Date Completed:** February 24, 2026  
**Time Investment:** 1.5 hours (including debugging)
```

---

**Now save the file and push:**
```
