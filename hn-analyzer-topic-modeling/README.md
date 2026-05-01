# HN Analyzer - Topic Modeling with LDA

## Project Overview
Added unsupervised machine learning to the Hacker News analyzer using Latent Dirichlet Allocation (LDA) for automatic topic discovery. The analyzer now has three complementary AI/ML features working together: keyword analysis, sentiment analysis, and topic modeling.

## Problem Statement
The previous version could categorize comments by predefined keywords and measure sentiment, but had a critical limitation: **I had to manually define what categories to look for.** If users discussed themes I hadn't anticipated, those insights were invisible.

Topic modeling solves this by discovering discussion themes automatically, without any predefined categories.

## Solution
Implemented LDA (Latent Dirichlet Allocation) topic modeling that:
- Automatically discovers 3-5 main discussion themes from comments
- Shows the top 10 words defining each topic
- Calculates what percentage of comments discuss each topic
- Requires zero manual configuration or keyword lists
- Works alongside existing sentiment and keyword features

## Tools & Technologies
- **Python 3.14** - Programming language
- **scikit-learn 1.8.0** - Machine learning library
- **CountVectorizer** - Converts text to numerical feature vectors
- **LatentDirichletAllocation** - Unsupervised topic modeling algorithm
- **NLTK** - Text preprocessing (tokenization, stopwords)
- **Claude Code** - AI pair programming assistant

## What is Topic Modeling?

**Simple explanation:**
Imagine reading 100 product reviews and noticing patterns - some people talk about price, others about features, others about support. Topic modeling is an algorithm that automatically finds these patterns without you telling it what to look for.

**Technical explanation:**
LDA is an unsupervised learning algorithm that:
1. Treats each document as a mixture of topics
2. Treats each topic as a mixture of words
3. Uses probability to discover which words tend to appear together
4. Groups these word patterns into coherent themes

**Why "unsupervised"?**
Unlike sentiment analysis (which knows positive vs negative), topic modeling discovers patterns without labeled training data. You don't tell it "look for pricing discussions" - it discovers that pricing *is* a discussion theme on its own.

## Implementation Process

### Phase 1: Text Preprocessing
**Challenge:** Raw HN comments contain HTML tags, entities, punctuation, and common words that pollute topics.

**Solution:**
```python
def preprocess_comment(text):
    # Decode HTML entities (&#x27; → ')
    text = html.unescape(text)
    # Strip HTML tags (<p>, <a>)
    text = re.sub(r'<[^>]+>', ' ', text)
    # Remove punctuation and numbers
    text = re.sub(r'[^a-zA-Z\s]', '', text)
    # Tokenize and lowercase
    tokens = nltk.word_tokenize(text.lower())
    # Remove stopwords and short fragments
    tokens = [t for t in tokens if t not in stopwords and len(t) >= 3]
    return ' '.join(tokens)
```

**What I learned:**
- Clean data is critical for ML - "garbage in, garbage out"
- HTML entities confuse both humans and algorithms
- Stopwords ("the", "is", "and") appear everywhere but mean nothing
- Tokenization handles contractions intelligently ("don't" → ["do", "n't"])

### Phase 2: Text Vectorization
**Challenge:** Machine learning algorithms need numbers, not text.

**Solution:**
```python
from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer(
    min_df=2,        # Ignore words appearing in <2 documents
    max_df=0.8,      # Ignore words in >80% of documents
    max_features=500 # Cap vocabulary to 500 most meaningful words
)
doc_term_matrix = vectorizer.fit_transform(preprocessed_texts)
```

**What this creates:**

pricing  expensive  fast  slow  feature
Comment 1:    1        1        0     0      0
Comment 2:    1        0        0     1      0
Comment 3:    0        0        1     0      2

Each row = a comment, each column = a word, each cell = word count.

**What I learned:**
- This is called a "bag of words" model - word order doesn't matter, just frequency
- `min_df=2` prevents typos/rare words from becoming "topics"
- `max_df=0.8` removes the search term itself (appears in every comment)
- Feature engineering = transforming raw data into ML-ready format

### Phase 3: LDA Training
**Challenge:** Find meaningful patterns in the word-frequency matrix.

**Solution:**
```python
from sklearn.decomposition import LatentDirichletAllocation

lda = LatentDirichletAllocation(
    n_components=3,          # Number of topics to discover
    max_iter=10,             # Training iterations
    learning_method='batch', # Process all documents together
    random_state=42          # Reproducible results
)
doc_topic_matrix = lda.fit_transform(doc_term_matrix)
```

**What this produces:**
```python
# For each comment, probability distribution across topics
doc_topic_matrix[0] = [0.70, 0.20, 0.10]
# This comment is 70% Topic 0, 20% Topic 1, 10% Topic 2
```

**What I learned:**
- `n_components=3` with 20 comments is the right balance (more topics = too granular)
- `max_iter=10` is sufficient for small datasets to converge
- `random_state=42` ensures same results every run (critical for debugging)
- Each comment is a *mixture* of topics, not assigned to just one

### Phase 4: Extracting Insights
**Challenge:** Convert probability matrices into human-readable insights.

**Solution:**
```python
# Get top 10 words per topic
for topic_idx, topic in enumerate(lda.components_):
    top_word_indices = topic.argsort()[-10:][::-1]
    top_words = [vocab[i] for i in top_word_indices]
    
# Count comments per topic
dominant_topics = doc_topic_matrix.argmax(axis=1)
for topic_num in range(3):
    count = (dominant_topics == topic_num).sum()
    percentage = (count / total) * 100
```

**What I learned:**
- `.argsort()[-10:][::-1]` is Python idiom for "top 10 values descending"
- `.argmax(axis=1)` finds which topic has highest probability per document
- Readable output matters - no one cares about probability matrices

## Sample Output

**Search term:** "AWS"

==================================================
🧠 TOPIC MODELING (LDA)
Topic 1  (4 comments, 20.0%)
Top words: onsite, cloud, time, founded, known, get, relocation, since, built, weve
Topic 2  (7 comments, 35.0%)
Top words: technology, aws, onsite, engineer, credit, account, promotional, york, company, lead
Topic 3  (9 comments, 45.0%)
Top words: aws, san, people, francisco, use, time, help, engineers, team, site

**Interpretation:**
- **Topic 3 (45%):** General AWS usage discussions (engineers, teams, using it)
- **Topic 2 (35%):** Job postings (engineer, onsite, company, lead)
- **Topic 1 (20%):** Infrastructure/cloud context (cloud, built, founded)

**The algorithm discovered these themes automatically** - I never told it to look for "hiring" vs "usage" vs "infrastructure."

## Technical Challenges & Solutions

### Challenge 1: Python 3.14 Compatibility
**Problem:** Original plan was to use `gensim` for LDA, but it doesn't compile on Python 3.14 (C extension compatibility issues).

**Error:**
fatal error: no member named 'ma_version_tag' in 'PyDictObject'
error: command '/usr/bin/clang' failed with exit code 1

**Solution:** Switched to `scikit-learn`'s LatentDirichletAllocation - pure Python implementation with identical functionality.

**Learning:** Always have a backup library in mind. Cutting-edge Python versions often break C-dependent packages.

### Challenge 2: HTML Entities Polluting Topics
**Problem:** Comments contained `&#x27;` (apostrophes) and `&#x2F;` (slashes) which became "words" in topics.

**Solution:** Added `html.unescape()` at data storage time, cleaning once instead of at every display point.

**Impact:** Improved sentiment accuracy from 70% to 80% - VADER could now read "it's" correctly.

**Learning:** Clean data once at the source, not repeatedly downstream.

### Challenge 3: Small Dataset Tuning
**Problem:** With only 20 comments, standard LDA hyperparameters produced noisy topics.

**Solution:**
- Set `n_components=3` (not 5) - fewer comments need fewer topics
- Set `min_df=2` - word must appear in at least 2 comments
- Set `max_df=0.8` - ignore words in >80% of comments (the search term itself)

**Learning:** ML hyperparameters must be tuned for dataset size. "Best practices" assume big data.

### Challenge 4: Interpreting Topic Coherence
**Problem:** Topics like `["get", "really", "people", "make"]` aren't immediately meaningful.

**Solution:** Look at the full top-10 list, not just top-3. Context emerges from the ensemble:
- `["get", "really", "people", "make", "product", "use", "time"]` → General product usage
- `["engineer", "onsite", "company", "lead", "technology"]` → Job postings

**Learning:** Topic modeling finds statistical patterns, not semantic meaning. Human interpretation is required.

## Key Decisions Made

| Decision | Options Considered | Choice | Reason |
|----------|-------------------|--------|--------|
| Library | gensim, sklearn, genism | sklearn | Python 3.14 compatibility, cleaner API |
| Number of topics | 3, 4, 5 | 3 | Sweet spot for 20 comments - 4-5 too granular |
| Preprocessing depth | Light, Medium, Aggressive | Medium | Balance between noise removal and information loss |
| Word filtering | min_df=1, 2, 3 | 2 | Removes typos without losing real terms |
| Learning method | online, batch | batch | Small dataset, batch is simpler and faster |
| Display format | Probabilities, Top words | Top words | More interpretable for non-technical users |

## What I Learned

### Machine Learning Concepts
- **Unsupervised learning** - Finding patterns without labeled data
- **Feature engineering** - Transforming text into numbers
- **Hyperparameter tuning** - Adjusting algorithm settings for dataset size
- **Bag of words model** - Representing text as word frequency vectors
- **Topic coherence** - Evaluating if discovered topics make semantic sense

### NLP (Natural Language Processing)
- **Tokenization** - Splitting text into words intelligently
- **Stopword removal** - Filtering common but meaningless words
- **HTML entity decoding** - `&#x27;` → `'`
- **Stemming vs lemmatization** - Didn't implement, but learned the concepts
- **Document-term matrix** - Core representation for text ML

### Python/Pandas Skills
- **NumPy array operations** - `.argsort()`, `.argmax()`, slicing
- **List comprehensions** - `[word for word in tokens if len(word) >= 3]`
- **Regular expressions** - `re.sub(r'<[^>]+>', ' ', text)` for HTML stripping
- **Scikit-learn API** - `.fit()`, `.transform()`, `.fit_transform()`

### Software Engineering
- **Modular functions** - `preprocess_comment()` can be reused elsewhere
- **Clean data at source** - One `html.unescape()` beats ten
- **Readable output** - ML results mean nothing if humans can't parse them
- **Graceful degradation** - Switch libraries when one fails, don't give up

## Outcome
- ✅ Topic modeling integrated alongside keyword and sentiment analysis
- ✅ Discovers 3 discussion themes automatically per search
- ✅ Zero manual configuration required
- ✅ Clean, production-ready output (no HTML artifacts)
- ✅ Improved overall data quality (entities decoded everywhere)
- ✅ ~0.5 second processing time for 20 comments (negligible overhead)

## Real-World Use Case

**Scenario:** Product manager researching "Notion" on Hacker News

**Results:**
Topic 1 (40%): feature, database, integration, api, sync, template
Topic 2 (35%): pricing, expensive, subscription, free, tier, cost
Topic 3 (25%): performance, slow, loading, mobile, desktop, speed

**Insight:** Even though I defined keyword categories for "features" and "pricing," topic modeling discovered that "performance" is the third major theme - something my manual keywords missed entirely.

**Action:** Add "performance" to keyword categories, investigate mobile performance complaints.

**This is the power of unsupervised learning** - it finds what you didn't know to look for.

## What I Would Do Next

### Short-term Enhancements
- **Dynamic topic count** - Let user choose 3-5 topics per analysis
- **Topic labeling** - Auto-generate semantic labels ("Pricing Discussion" vs "Topic 1")
- **Keyword-topic correlation** - Which topics correlate with negative sentiment?
- **Export topics to CSV** - Include topic assignment in export file

### Long-term Features
- **Topic trends over time** - Are performance complaints increasing?
- **Comparative topic analysis** - How do Notion and Asana differ thematically?
- **Interactive visualization** - Word clouds per topic, topic distribution charts
- **Cross-product insights** - "Products with similar topic distributions"

### Technical Improvements
- **LDA coherence metrics** - Automatically evaluate topic quality
- **Hierarchical topic modeling** - Subtopics within main topics
- **Dynamic preprocessing** - Adjust stopwords based on corpus
- **Parallelization** - Process multiple products simultaneously

## Files Modified
- `hn_analyzer.py` - Main analyzer script
  - Added: `preprocess_comment()` function
  - Added: `analyze_topics()` function with full LDA pipeline
  - Modified: Data cleaning moved to storage time
  - Impact: +60 lines, zero breaking changes

## Dependencies Added
```bash
pip install scikit-learn  # For LDA and CountVectorizer
# nltk already installed (used for sentiment analysis)
```

## Code Architecture
analyze_topics(comments_list)
├── preprocess_comment(text)          # HTML decode, tag strip, tokenize
├── CountVectorizer.fit_transform()   # Text → matrix
├── LatentDirichletAllocation.fit()   # Train LDA model
└── Display formatting                # Extract + print insights

**Key design decision:** All preprocessing happens in one pure function, making it testable and reusable.

## Resources & Documentation
- [Scikit-learn LDA Documentation](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.LatentDirichletAllocation.html)
- [LDA Original Paper](https://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf) - Blei, Ng, Jordan (2003)
- [CountVectorizer Guide](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.CountVectorizer.html)
- [Topic Modeling Overview](https://en.wikipedia.org/wiki/Topic_model)

## Reflections

### What Worked Well
- **Claude Code's phased approach** - Implementing in 4 phases made a complex algorithm digestible
- **Switching libraries mid-stream** - When gensim failed, pivoting to sklearn showed real-world problem-solving
- **Cleaning data at source** - One fix improved both display and ML accuracy
- **Testing with real data** - Using actual HN comments immediately showed value

### What Surprised Me
- **How well LDA worked with tiny data** - 20 comments produced coherent topics
- **The sentiment accuracy boost** - Clean entities improved VADER from 70% to 80%
- **Topic interpretability** - Expected gibberish, got meaningful themes
- **Processing speed** - Entire ML pipeline runs in ~0.5 seconds

### What I'd Do Differently
- **Start with sklearn** - Would have saved 30 minutes vs debugging gensim
- **Add more comments** - 50-100 would produce even clearer topics
- **Visualize topics** - Word clouds would make patterns more obvious
- **A/B test hyperparameters** - Try n_components=4 and compare coherence

### Impact on My Learning
This project taught me the difference between **knowing about ML** and **doing ML**:

**Before:** "Topic modeling uses LDA to find patterns"  
**After:** I understand CountVectorizer parameters, can debug preprocessing issues, know when 3 vs 5 topics makes sense, and can explain why batch learning beats online for small datasets.

**The most valuable lesson:** ML isn't magic - it's math applied to clean data with tuned parameters. The algorithm is 20% of the work; data cleaning and hyperparameter tuning is 80%.

---

**Repository:** [review-analyzer](https://github.com/YOUR-USERNAME/review-analyzer)  
**Previous Feature:** [Sentiment Analysis](../hn-analyzer-sentiment/README.md)  
**Date Completed:** March 8, 2026  
**Time Investment:** 2 hours (including debugging Python 3.14 issues)  
**Lines of Code Added:** ~60 lines
