# HN Analyzer - Data Visualizations

## Project Overview
Transformed the command-line HN analyzer into a visual analytics platform by adding professional data visualizations using matplotlib. The tool now generates both detailed text reports and shareable visual insights in a single run.

## Problem Statement
The previous version provided rich text-based analysis (keywords, sentiment, topics, TF-IDF), but had critical limitations:
- **Hard to share insights** with non-technical stakeholders
- **Difficult to spot patterns** in pure text format
- **No quick visual summary** for presentations or reports
- **Required manual data interpretation** from CSV exports

Product managers need to communicate insights quickly and visually - charts beat tables every time.

## Solution
Added automated visualization generation that creates a 2x2 professional chart layout showing:
1. **Sentiment distribution** - Visual breakdown of positive/neutral/negative
2. **Topic clustering** - Which themes dominate the discussion
3. **Key terms** - Most distinctive vocabulary with scores
4. **Category comparison** - Predefined keyword category analysis

All charts save automatically as a high-quality PNG alongside the CSV export.

## Tools & Technologies
- **matplotlib 3.10.9** - Python's standard visualization library
- **NumPy 2.4.4** - Numerical computing for data manipulation
- **seaborn-v0_8-whitegrid style** - Clean, professional aesthetic
- **Python's built-in color schemes** - Green/red for sentiment, gradients for scores

## Implementation Details

### Architecture Decision
Rather than re-computing analysis for visualization, modified existing functions to return their results:

```python
# Before: functions only printed, didn't return
analyze_topics(comments)  # Printed topics, returned nothing

# After: functions return data for reuse
topic_counts, top_words = analyze_topics(comments)
```

**Benefits:**
- ✅ Compute once, use twice (terminal + visualization)
- ✅ Cleaner code architecture
- ✅ Easier to test individual components
- ✅ More flexible for future features

### Chart Specifications

#### 1. Sentiment Pie Chart (Top-Left)
```python
colors = ['#2ecc71', '#95a5a6', '#e74c3c']  # Green/Gray/Red
plt.pie(values, labels=labels, autopct='%1.1f%%', colors=colors)
```

**Design choices:**
- Green = positive (universally understood)
- Red = negative (universally understood)
- Gray = neutral (de-emphasized)
- Auto-suppresses 0% slices (cleaner)
- Percentage labels directly on slices

**Why pie chart?**
Three-category distributions are perfect for pie charts - shows the whole at a glance.

#### 2. Topic Distribution Bar Chart (Top-Right)
```python
bars = ax.barh(topic_labels, counts, color=plt.cm.Blues(norm_counts))
```

**Design choices:**
- Horizontal bars (easier to read topic previews)
- Blue gradient intensity = comment count
- Topic word previews as labels (not just "Topic 1")
- Count annotations on bars

**Why horizontal?**
Topic labels can be long ("amazon, services, web...") - horizontal fits better.

#### 3. TF-IDF Key Terms (Bottom-Left)
```python
colors = plt.cm.viridis(scores / max(scores))
ax.barh(terms, scores, color=colors)
```

**Design choices:**
- Horizontal bars ranked top-to-bottom (#1 at top)
- Color intensity proportional to score (darker = more important)
- Viridis colormap (colorblind-friendly)
- Score values visible on x-axis

**Why color gradient?**
Makes it immediately obvious which terms are most distinctive.

#### 4. Keyword Categories (Bottom-Right)
```python
filtered_cats = {k: v for k, v in keyword_counts.items() if v > 0}
ax.bar(categories, counts, color='#3498db')
```

**Design choices:**
- Vertical bars (short category names)
- Filters zero-count categories (cleaner output)
- Solid blue (simpler than gradient for comparison)
- Count labels above bars

**Why filter zeros?**
With 6 categories, showing 3 empty bars clutters the chart.

### Layout & Styling

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))
plt.style.use('seaborn-v0_8-whitegrid')
fig.suptitle(f"HN Analysis: {search_term}", fontsize=14, fontweight='bold')
plt.tight_layout()
plt.savefig(filename, dpi=150, bbox_inches='tight')
```

**Key parameters:**
- `figsize=(14, 10)` - Large enough for readability, small enough for emails
- `dpi=150` - High resolution for presentations
- `bbox_inches='tight'` - Removes excess whitespace
- `seaborn-v0_8-whitegrid` - Professional, minimal style

## Sample Output

**Search term:** "AWS"

**File generated:** `hn_AWS_analysis.png`

**What it shows:**
- **Sentiment:** 85% positive, 5% negative, 10% neutral (pie chart)
- **Topics:** 45% general usage, 35% hiring, 20% infrastructure (bars)
- **Key terms:** "engineer", "cloud", "onsite", "services" (gradient bars)
- **Categories:** Support 25%, Pricing 10%, Performance 5% (comparison bars)

**Time to generate:** ~1 second (after text analysis completes)

**Use case:** PM can now drop this PNG into a stakeholder deck showing "What developers say about AWS on HN"

## Technical Challenges & Solutions

### Challenge 1: matplotlib Not Installed
**Problem:** First run failed - matplotlib isn't part of Python standard library.

**Solution:**
```bash
pip3 install matplotlib numpy
```

**Learning:** Always check dependencies before running visualization code.

### Challenge 2: Functions Didn't Return Data
**Problem:** `analyze_topics()` and `extract_key_terms()` only printed results, visualization had no data to work with.

**Solution:** Modified both functions to return their computed values:
```python
# analyze_topics now returns
return topic_counts, top_words_per_topic

# extract_key_terms now returns  
return top_terms
```

**Impact:** Had to update main flow to capture these returns, but resulted in cleaner architecture overall.

**Learning:** When designing functions, always consider if other code might need the results. Return data even if you're also printing it.

### Challenge 3: Neutral Sentiment at 0%
**Problem:** Pie chart showed ugly 0% slice for neutral when no neutral comments exist.

**Solution:**
```python
# Filter out zero values before plotting
non_zero_sentiments = {k: v for k, v in sentiment_counts.items() if v > 0}
```

**Learning:** Always filter zero values in visualizations - they clutter without adding information.

### Challenge 4: Topic Labels Too Long
**Problem:** Topic labels like "amazon, services, product, web, get..." don't fit on vertical bars.

**Solution:** Used horizontal bars (`barh`) instead of vertical (`bar`).

**Learning:** Chart orientation should match label length. Long labels = horizontal bars.

### Challenge 5: Color Accessibility
**Problem:** Default matplotlib colors can be hard to distinguish for colorblind users.

**Solution:** Used colorblind-friendly schemes:
- Viridis colormap for gradients
- Explicit green/red only for sentiment (universally understood context)
- Blue for neutral categories

**Learning:** Accessibility isn't optional. Viridis/plasma/cividis are safe defaults.

## Key Decisions Made

| Decision | Options Considered | Choice | Reason |
|----------|-------------------|--------|--------|
| Visualization library | matplotlib, seaborn, plotly | matplotlib | Standard, no dependencies on web frameworks |
| Chart layout | 1x4, 2x2, 4x1 | 2x2 | Balanced, fits on one screen |
| File format | PNG, PDF, SVG | PNG | Universal compatibility, good for email/Slack |
| Color scheme | Default, seaborn, custom | seaborn whitegrid | Professional, minimal, accessible |
| DPI | 72, 100, 150, 300 | 150 | Sharp enough for presentations, small file size |
| Return data or not | Print only, Return only, Both | Both | Terminal output + reusable data |

## What I Learned

### Data Visualization Principles
- **Choose chart type by data structure** - 3 categories = pie, comparisons = bars, rankings = horizontal bars
- **Color has meaning** - Green/red for sentiment, gradients for scores, solid for categories
- **Filter noise** - Zero values, duplicate labels, excess decimals all clutter
- **Accessibility matters** - Colorblind-friendly palettes aren't optional
- **Labels are critical** - "Topic 1" tells you nothing; "amazon, services, web..." tells a story

### matplotlib API
- `plt.subplots(2, 2)` creates grid layouts
- `barh()` for horizontal bars, `bar()` for vertical
- `autopct='%1.1f%%'` formats pie chart percentages
- `plt.cm.Blues()` accesses color maps
- `tight_layout()` prevents label overlap
- `savefig(dpi=150)` controls image quality

### Software Architecture
- **Separation of concerns** - Analysis functions compute, visualization functions display
- **Return values > side effects** - Functions that return data are more reusable
- **Single source of truth** - Compute once, use many times
- **Progressive enhancement** - Added visualization without breaking existing features

### Product Management Insights
- **Visuals communicate faster** - A pie chart beats "80% positive" for stakeholders
- **Multiple formats serve different audiences** - CSV for analysts, PNG for executives, terminal for developers
- **Professional polish matters** - Seaborn whitegrid looks infinitely better than matplotlib defaults
- **Shareable = valuable** - PNG in Slack beats "run this Python script"

## Outcome
- ✅ 4 professional charts generated automatically
- ✅ High-resolution PNG (150 DPI) suitable for presentations
- ✅ Colorblind-friendly color schemes
- ✅ Zero manual work - runs with every analysis
- ✅ File naming convention: `hn_{product}_analysis.png`
- ✅ Processing time: <1 second additional overhead
- ✅ No breaking changes to existing features

## Real-World Use Case

**Scenario:** PM preparing competitive analysis presentation

**Before visualization:**
1. Run analyzer (get CSV)
2. Open Excel
3. Create charts manually
4. Export images
5. Insert into deck
**Time:** 15-20 minutes

**After visualization:**
1. Run analyzer (get PNG)
2. Drag PNG into deck
**Time:** 30 seconds

**Impact:** 30x faster turnaround for stakeholder-ready insights.

## Sample Stakeholder Use Cases

### For Executives
**Question:** "How do developers feel about our competitor?"  
**Answer:** [Show sentiment pie chart] "85% positive sentiment in discussions"

### For Product Teams
**Question:** "What do people actually talk about when discussing Product X?"  
**Answer:** [Show topic distribution] "45% usage discussions, 35% hiring, 20% infrastructure"

### For Marketing
**Question:** "What makes our product distinctive in conversations?"  
**Answer:** [Show TF-IDF chart] "The terms 'integration', 'workflow', 'automation' appear most"

### For Sales
**Question:** "Are pricing complaints a major issue?"  
**Answer:** [Show keyword categories] "Pricing mentioned in only 10% of comments"

## What I Would Do Next

### Enhanced Visualizations
- **Sentiment over time** - Line chart showing sentiment trends
- **Word clouds** - Visual representation of TF-IDF terms
- **Comparison view** - Side-by-side charts for 2 products
- **Interactive charts** - Plotly for drill-down capabilities

### Export Options
- **PDF report** - Multi-page document with all charts + text
- **PowerPoint export** - Auto-generate presentation slides
- **HTML dashboard** - Interactive web view
- **Email integration** - Auto-send visualizations to stakeholders

### Advanced Analytics
- **Correlation heatmap** - Which topics correlate with sentiment?
- **Network graph** - How do topics relate to each other?
- **Trend arrows** - Up/down indicators for change over time
- **Benchmark comparison** - How does this product compare to category average?

## Files Modified
- `hn_analyzer.py`
  - Added: `generate_visualizations()` function (+69 lines)
  - Modified: `analyze_topics()` to return data
  - Modified: `extract_key_terms()` to return data
  - Modified: Main flow to capture returns and call visualization
  - Added: matplotlib, numpy imports

## Dependencies Added
```bash
pip3 install matplotlib numpy
```

**Package versions:**
- matplotlib 3.10.9
- numpy 2.4.4
- (Plus transitive dependencies: pillow, fonttools, etc.)

## Output Files Per Run
hn_{product}comments.csv         # Data export (unchanged)
hn{product}_analysis.png          # NEW: Visual report

Both files use same naming convention for easy matching.

## Code Quality Improvements

**Before visualization:**
```python
def analyze_topics(comments_list):
    # ... compute topics ...
    # Print results
    # No return value
```

**After visualization:**
```python
def analyze_topics(comments_list):
    # ... compute topics ...
    # Print results
    return topic_counts, top_words_per_topic  # Now reusable!
```

**Impact:** Functions are now testable, reusable, and follow single-responsibility principle.

## Reflections

### What Worked Well
- **2x2 layout** - Perfect balance of information density
- **Seaborn style** - Instantly made charts look professional
- **Color choices** - Green/red for sentiment is universally understood
- **Auto-filtering zeros** - Cleaner output without manual intervention
- **High DPI** - Charts look sharp in presentations

### What Surprised Me
- **How fast matplotlib renders** - <1 second for 4 charts
- **How much color matters** - Same data, better colors = 10x more readable
- **The value of horizontal bars** - Dramatically improved topic chart readability
- **Font cache on first run** - matplotlib builds a font database initially (one-time delay)

### What I'd Do Differently
- **Add chart titles** - Each subplot could use a clearer title
- **Experiment with size** - 14x10 works but 16x12 might be better for projectors
- **Add gridlines to bars** - Makes values easier to read
- **Test with more products** - Only validated with Amazon/AWS/Notion

### Impact on My Learning
This was my first data visualization project. Key takeaway: **Good charts tell a story without explanation.**

Before this, I thought visualization was about "making things pretty." Now I understand it's about **choosing the right representation for the data structure and user goal.**

- Pie charts for parts-of-whole (sentiment distribution)
- Bar charts for comparisons (topics, categories)  
- Horizontal bars for rankings (TF-IDF)
- Color gradients for continuous values (scores)

**The visualization should make the insight obvious, not require a legend and explanation.**

---

**Repository:** [review-analyzer](https://github.com/YOUR-USERNAME/review-analyzer)  
**Previous Feature:** [Topic Modeling](../hn-analyzer-topic-modeling/README.md)  
**Date Completed:** March 8, 2026  
**Time Investment:** 1.5 hours (including matplotlib setup)  
**Lines of Code Added:** ~80 lines