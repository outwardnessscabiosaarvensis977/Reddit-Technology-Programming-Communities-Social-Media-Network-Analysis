# Reddit Technology & Programming Communities: Network and Text Analytics

## Executive Summary

This project analyzes how users interact and what they discuss across technology and programming communities on Reddit. It combines **social network analysis**, **natural language processing (NLP)**, **sentiment analysis**, and **topic modeling** to convert Reddit discussion data into interpretable community and conversation insights.

The source dataset was collected from technology and programming-related subreddits using the **PullPush Reddit API**. The raw dataset contains **6,225 comments across 28 subreddits, 680 posts, and 3,211 unique authors**. After preprocessing, the analytical dataset contains **5,437 usable comments, 549 posts, and 3,103 unique authors**, covering **4 March 2025 to 19 May 2025**.

A directed weighted user-reply network was constructed from comment-to-comment replies. The final network contains **1,692 users and 1,991 directed weighted reply relationships**, representing **2,366 aggregated reply interactions**. Centrality measures were used to distinguish highly replied-to users, active responders, structurally influential users, and bridge users. Louvain community detection identified **278 interaction communities** with a **modularity score of 0.953**, indicating strong structural separation between discussion groups.

The text-analysis component compares **VADER** with the transformer-based **RoBERTa** sentiment model, applies **TF-IDF** keyword analysis, and uses **Latent Dirichlet Allocation (LDA)** to identify six major discussion topics. RoBERTa classified **51.37% of comments as neutral**, **29.67% as negative**, and **18.96% as positive**, which is consistent with the explanatory, troubleshooting, and technical nature of many technology discussions.

The project demonstrates how network structure and conversation content can be combined to support **community intelligence, developer-relations analysis, product/support research, content strategy, and technology ecosystem monitoring**. These applications are decision-support interpretations of the analysis rather than claims that Reddit activity directly represents customer satisfaction, market demand, or business performance.

---

## Business / Decision Problem

Organizations operating in technology ecosystems often need to understand large volumes of online discussion without relying only on comment counts or manual reading.

Developer-relations teams, community managers, product and support analysts, research teams, and technology organizations may want to understand:

- Where discussion activity is concentrated.
- Which interaction groups form naturally within a wider community.
- Which users occupy structurally important roles in conversations.
- Which users connect otherwise separated discussion groups.
- What technical topics dominate discussion.
- Whether discussion tone differs across subreddits or interaction communities.
- Which communities are associated with troubleshooting, criticism, debate, or more constructive discussion.
- How network structure and discussion content can be combined to identify meaningful community segments.

A simple volume analysis cannot answer these questions because it does not show **who interacts with whom** or **what connected groups are discussing**.

This project addresses that gap by integrating network analytics with NLP.

---

## Business Analysis Questions

From a decision-support perspective, the project can be used to explore questions such as:

1. Which technology and programming communities generate the highest discussion activity?
2. Which users attract the most replies and which users participate most actively?
3. Which users act as bridges between otherwise separated discussion groups?
4. Are Reddit technology discussions organized as one connected ecosystem or many smaller interaction clusters?
5. Which themes dominate technology and programming conversations?
6. How does sentiment vary across subreddits and detected user communities?
7. Do structurally detected communities also have identifiable discussion topics?
8. How could community segmentation support more targeted research, support, education, or engagement strategies?

---

## Analytical Objectives

The project was designed to:

- Collect Reddit submissions and comments from selected technology and programming communities.
- Validate and clean comment, author, reply, and timestamp data.
- Distinguish post replies from direct comment-to-comment replies.
- Construct a directed weighted user-reply network.
- Measure different forms of user centrality and participation.
- Detect network communities using the Louvain method.
- Evaluate community separation using modularity.
- Compare lexicon-based and transformer-based sentiment analysis.
- Identify distinctive terms using TF-IDF.
- Identify recurring discussion themes using LDA topic modeling.
- Connect Louvain communities with sentiment and dominant topics.
- Translate structural and textual findings into interpretable stakeholder insights.

---

## Data Source and Collection

The data was collected from Reddit using the **PullPush Reddit API**.

Two API endpoints are used by the data-collection notebook:

- Reddit submission search
- Reddit comment search

The collection workflow is implemented in:

`notebooks/fetch_selected_reddit_comments.ipynb`

The notebook collects fields including:

- `subreddit`
- `post_id`
- `post_title`
- `comment_id`
- `author`
- `body`
- `parent_id`
- `link_id`
- `score`
- `created_utc`
- `created_datetime`

The collected raw dataset is stored as:

`data/reddit_technology_programming_comments.csv`

### Raw Dataset Summary

| Metric | Value |
|---|---:|
| Comments | 6,225 |
| Subreddits | 28 |
| Posts | 680 |
| Unique authors | 3,211 |
| Unique comment IDs | 6,123 |
| Duplicate comment IDs identified | 102 |

### Cleaned Analytical Dataset

After duplicate, deleted, removed, invalid, and unusable records are handled:

| Metric | Value |
|---|---:|
| Usable comments | 5,437 |
| Subreddits | 28 |
| Unique posts | 549 |
| Unique authors | 3,103 |
| Comment-to-comment replies | 3,056 |
| Direct replies to posts | 2,381 |
| Start date | 4 March 2025 |
| End date | 19 May 2025 |

The final cleaned dataset contains no duplicate comment IDs and no missing values in the main analytical fields used by the project.

---

## Analytical Workflow

```text
Reddit / PullPush Data Collection
                |
                v
      Raw Comment Dataset
                |
                v
Data Validation and Preprocessing
                |
        +-------+-------+
        |               |
        v               v
 User Reply Network   Cleaned Text
        |               |
        v               v
Centrality Analysis   VADER / RoBERTa
        |               |
        v               v
Louvain Communities   TF-IDF / LDA
        |               |
        +-------+-------+
                |
                v
 Community + Topic + Sentiment Integration
                |
                v
 Business / Stakeholder Interpretation
```

---

## Methodology

### 1. Data Preprocessing

The main analysis notebook performs data-quality checks and preprocessing before network or NLP analysis.

The process includes:

- Removing duplicate comment IDs.
- Removing deleted authors.
- Removing deleted or removed comments.
- Removing invalid or empty text.
- Excluding AutoModerator content where appropriate.
- Converting Unix timestamps to readable datetime values.
- Preparing cleaned text for NLP.
- Identifying whether a record is a reply to a post (`t3_`) or another comment (`t1_`).

Only comment-to-comment replies are used to create direct user-to-user network edges because both the source user and target user can be identified.

---

### 2. Exploratory Analysis

The cleaned dataset contains 5,437 comments across 28 subreddits, but activity is unevenly distributed.

The five largest subreddits by cleaned comment volume are:

| Subreddit | Comments | Posts | Authors |
|---|---:|---:|---:|
| GenAI4all | 863 | 40 | 534 |
| datascience | 772 | 29 | 465 |
| java | 502 | 29 | 256 |
| technology | 475 | 20 | 321 |
| linux | 454 | 18 | 282 |

This imbalance is important because high-volume communities can have greater influence on overall sentiment and topic-modeling results.

---

### 3. User Reply Network Construction

The project models Reddit interactions as a **directed weighted network**.

- **Node:** Reddit user
- **Directed edge:** Reply from one user to another
- **Source:** User writing the reply
- **Target:** Author of the parent comment
- **Edge weight:** Number of replies between the same source and target
- **Network type:** Directed weighted user-reply network

The final network is exported as:

`network/user_reply_network.graphml`

### Network Summary

| Network Metric | Result |
|---|---:|
| User nodes | 1,692 |
| Directed weighted edges | 1,991 |
| Aggregated reply interactions | 2,366 |
| Density | 0.000696 |
| Weakly connected components | 264 |
| Strongly connected components | 1,242 |

The low density and large number of connected components indicate that discussions are highly fragmented, with users typically interacting inside smaller threads or discussion groups rather than one unified network.

---

### 4. Network Centrality Analysis

Multiple centrality measures are used because "importance" in a discussion network can represent different behaviors.

| Measure | Interpretation |
|---|---|
| In-degree | Users receiving replies from many others |
| Out-degree | Users replying to many others |
| Weighted in-degree | Users receiving repeated replies |
| Weighted out-degree | Users repeatedly responding to others |
| PageRank | Structurally influential users connected through important reply relationships |
| Betweenness centrality | Bridge users connecting otherwise separated parts of the network |

This distinction is important for business and community analysis because a user who attracts discussion is not necessarily the same user who connects separate groups.

---

### 5. Louvain Community Detection

The network is converted to an undirected representation for Louvain community detection.

Results:

- **278 detected communities**
- **Modularity: 0.953**
- **Largest community: 96 users**

The high modularity indicates strong separation between interaction groups.

Large communities are associated with technical contexts including programming, Java, GenAI, Linux, data, databases, AWS, code, and related discussion.

---

### 6. Sentiment Analysis

Two sentiment approaches are compared.

#### VADER

VADER is used as a rule/lexicon-based baseline model suitable for short social-media text.

#### RoBERTa

The main model is:

`cardiffnlp/twitter-roberta-base-sentiment-latest`

RoBERTa is used for final interpretation because its transformer architecture captures context more effectively than a lexicon-only model.

### Sentiment Comparison

| Model | Positive | Neutral | Negative |
|---|---:|---:|---:|
| VADER | 55.14% | 21.98% | 22.88% |
| RoBERTa | 18.96% | 51.37% | 29.67% |

The large difference between the models is analytically important. Technical comments frequently contain words related to problems, errors, fixes, limitations, or improvement without necessarily expressing strong emotional sentiment.

For this reason, sentiment should be interpreted together with topic and discussion context.

---

### 7. TF-IDF Keyword Analysis

TF-IDF is used to identify distinctive terms across the Reddit comments.

Prominent terms include:

- `ai`
- `data`
- `code`
- `work`
- `people`
- `time`
- `use`
- `like`

The keyword results confirm that the dataset contains a mixture of AI, programming, data, software, and general technology discussion.

---

### 8. LDA Topic Modeling

Latent Dirichlet Allocation is used to identify six recurring discussion themes.

| Topic | Representative Terms | Comments |
|---|---|---:|
| 1 | linux, java, windows | 602 |
| 2 | data, use, would | 855 |
| 3 | ai, work, time | 1,265 |
| 4 | people, ai, would | 1,107 |
| 5 | gt, lt, number | 342 |
| 6 | like, ai, things | 1,266 |

Topics 3 and 6 are the largest, reflecting broad AI and technology discussion.

Topic 4 has the most negative average RoBERTa sentiment among the six topics, suggesting more critical, argumentative, or problem-oriented discussion within that theme.

---

### 9. Linking Network Communities with Text

The project's main analytical value comes from integrating Louvain communities with sentiment and LDA topic assignments.

This allows the analysis to move from:

> "These users interact frequently"

to:

> "These users interact frequently, tend to discuss these themes, and show this general sentiment pattern."

Examples from the analysis include:

- Linux-related communities associated with operating-system and programming vocabulary.
- Java interaction communities associated with programming-language discussion.
- GenAI4all communities associated with broader AI-related discussion.
- Community-level sentiment varying across structurally different user groups.

This integration demonstrates that detected network communities are not only mathematical clusters; many can also be interpreted as meaningful discussion groups.

---

## Key Findings

1. **Technology discussions are structurally fragmented.**  
   The network density is only 0.000696 and contains many connected components, indicating that most interaction occurs within smaller local discussion structures.

2. **Detected communities are strongly separated.**  
   Louvain community detection produced 278 communities with modularity of 0.953.

3. **User influence is multidimensional.**  
   Highly replied-to users, active responders, PageRank leaders, and bridge users are not always the same people.

4. **Technical discussion is predominantly neutral when context is considered.**  
   RoBERTa classified 51.37% of comments as neutral.

5. **VADER and RoBERTa produce substantially different interpretations.**  
   VADER classified 55.14% of comments as positive, compared with only 18.96% using RoBERTa.

6. **AI, data, programming languages, operating systems, and technical problem-solving are major themes.**

7. **Network communities have identifiable textual characteristics.**  
   Louvain groups can be connected with distinct dominant topics and differing sentiment patterns.

---

## Business Interpretation

### 1. Community Segmentation

The high modularity and fragmented network indicate that technology discussions should not automatically be treated as one homogeneous audience.

**Potential decision-support value:**  
Developer-relations or community teams could analyze communities separately when designing technical education, community programs, documentation research, or support initiatives.

---

### 2. Community Role Identification

Centrality measures reveal different structural roles:

- users attracting responses,
- highly active responders,
- structurally influential users, and
- bridge users connecting groups.

**Potential decision-support value:**  
Community managers and researchers can distinguish between activity, visibility, and structural connectivity instead of using one engagement metric as a proxy for "influence."

Centrality should not be interpreted as personal authority, expertise, endorsement, or customer value without additional evidence.

---

### 3. Product and Support Intelligence

Negative sentiment in technical communities can reflect:

- troubleshooting,
- bug discussion,
- criticism,
- disagreement,
- security concerns,
- tool limitations, or
- implementation problems.

**Potential decision-support value:**  
Product or support analysts could combine negative/problem-oriented sentiment with topic analysis to identify areas requiring deeper qualitative investigation.

Sentiment alone should not be treated as a customer-satisfaction score.

---

### 4. Content and Developer-Education Strategy

Topic modeling identifies recurring themes around AI, data, operating systems, programming languages, and code-related discussion.

**Potential decision-support value:**  
Organizations could use these topic patterns as exploratory evidence when deciding which technical themes deserve further research, documentation, tutorials, FAQs, or educational content.

---

### 5. Community Health and Cross-Group Connectivity

Betweenness centrality identifies users who connect otherwise separated parts of the network.

**Potential decision-support value:**  
At an aggregate level, this can help community researchers understand whether information is likely to remain inside local clusters or move between different discussion groups.

---

### 6. Technology Ecosystem Monitoring

Combining topic, sentiment, subreddit, and network information provides more context than simple keyword monitoring.

**Potential decision-support value:**  
Research and strategy teams could use a similar framework to monitor changing technical interests, emerging discussion clusters, or shifts in problem-oriented conversation over time.

A production use case would require longer-term and more representative data collection.

---

## Stakeholder Use Cases

Potential stakeholders for this type of analysis include:

- Developer Relations teams
- Technical Community Managers
- Product Analytics teams
- Customer/Technical Support research teams
- Technology strategy and market-research teams
- Open-source community researchers
- Social-media and community analysts
- Data science and NLP teams

This project demonstrates an analytical framework rather than a production monitoring system.

---

## Project Structure

```text
reddit-technology-programming-network-analysis/
|
|-- README.md
|-- requirements.txt
|-- .gitignore
|
|-- data/
|   `-- reddit_technology_programming_comments.csv
|
|-- notebooks/
|   |-- fetch_selected_reddit_comments.ipynb
|   `-- technology_and_programming_communities_reddit_analysis.ipynb
|
|-- network/
|   `-- user_reply_network.graphml
|
|-- reports/
|   `-- technology_and_programming_communities_reddit_analysis_report.pdf
|
`-- private/                      # local only; excluded from GitHub
    |-- technology_and_programming_communities_reddit_analysis_ppt.pptx
    `-- technology_and_programming_communities_reddit_analysis_report.docx
```

Local development files such as `.venv/`, `private/`, and `project_structure.txt` are intentionally excluded from GitHub through `.gitignore`.

---

## Main Project Files

### `notebooks/fetch_selected_reddit_comments.ipynb`

Collects Reddit submissions and comments from the selected technology/programming communities through PullPush and exports the raw CSV dataset.

### `notebooks/technology_and_programming_communities_reddit_analysis.ipynb`

Performs:

- preprocessing,
- exploratory analysis,
- user-reply network construction,
- centrality analysis,
- Louvain community detection,
- VADER sentiment analysis,
- RoBERTa sentiment analysis,
- TF-IDF analysis,
- LDA topic modeling,
- community-topic integration,
- visualizations, and
- GraphML export.

### `data/reddit_technology_programming_comments.csv`

Raw dataset generated by the data-collection notebook.

### `network/user_reply_network.graphml`

Directed weighted reply network generated by the analysis notebook. It can be opened in tools such as Gephi for additional network inspection.

### `reports/technology_and_programming_communities_reddit_analysis_report.pdf`

Full analytical report containing methodology, tables, figures, interpretation, research-question answers, and limitations.

---

## Technologies Used

### Programming and Environment

- Python
- Jupyter Notebook

### Data Collection and Processing

- Requests
- pandas
- NumPy

### Network Analysis

- NetworkX
- Directed weighted graphs
- PageRank
- Betweenness centrality
- Louvain community detection
- Modularity analysis

### NLP and Machine Learning

- NLTK
- VADER
- Hugging Face Transformers
- RoBERTa
- scikit-learn
- TF-IDF
- Latent Dirichlet Allocation

### Visualization

- Matplotlib
- WordCloud
- NetworkX graph visualization

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/reddit-technology-programming-network-analysis.git
```

```bash
cd reddit-technology-programming-network-analysis
```

### 2. Create a virtual environment

Windows Command Prompt / PowerShell:

```bash
python -m venv .venv
```

Activate in Command Prompt:

```bash
.venv\Scripts\activate
```

Activate in Git Bash:

```bash
source .venv/Scripts/activate
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Launch Jupyter

```bash
jupyter notebook
```

or open the notebooks directly in VS Code.

---

## NLTK Resources

The analysis uses:

- `vader_lexicon`
- `stopwords`

The analysis notebook downloads these resources automatically.

If required, run:

```python
import nltk

nltk.download("vader_lexicon")
nltk.download("stopwords")
```

---

## RoBERTa Model

The main sentiment model is:

`cardiffnlp/twitter-roberta-base-sentiment-latest`

The model is downloaded from Hugging Face the first time the analysis is run.

An internet connection is required for the initial model download. The model is normally cached locally after the first successful download.

---

## Run Order

Run the notebooks in this order:

### Step 1 — Collect Reddit Data

Run:

`notebooks/fetch_selected_reddit_comments.ipynb`

Expected output:

`data/reddit_technology_programming_comments.csv`

### Step 2 — Run the Main Analysis

Run:

`notebooks/technology_and_programming_communities_reddit_analysis.ipynb`

Expected network output:

`network/user_reply_network.graphml`

The main notebook also generates the analytical tables and visualizations used in the report.

---

## Important Path Configuration

The uploaded notebook versions were originally written when the generated CSV and GraphML files were in the same working directory as the notebooks.

After reorganizing the GitHub repository into `data/`, `notebooks/`, and `network/`, update the notebook file paths so that they match the organized structure.

If the notebook kernel is running with `notebooks/` as its working directory, use:

### Data collection output

```python
"../data/reddit_technology_programming_comments.csv"
```

### Main analysis input

```python
"../data/reddit_technology_programming_comments.csv"
```

### GraphML output

```python
"../network/user_reply_network.graphml"
```

This keeps the repository portable and avoids computer-specific absolute paths.

---

## Limitations

The findings should be interpreted as **exploratory community and conversation analytics**, not as a complete representation of Reddit or the technology market.

Key limitations include:

- Only selected technology and programming-related subreddits were analyzed.
- The PullPush API may not capture comments that were deleted, removed, unavailable, or otherwise missed during collection.
- Data volume is uneven across subreddits, with communities such as GenAI4all, datascience, java, technology, and linux contributing more comments than smaller communities.
- Aggregate topic and sentiment results can therefore be influenced by high-volume subreddits.
- RoBERTa can still misclassify sarcasm, informal language, technical disagreement, and code-like text.
- VADER is particularly sensitive to word-level polarity and provides a very different sentiment distribution from RoBERTa.
- The reply network is sparse, so some Louvain communities may represent short-lived thread-level groups rather than persistent social communities.
- The dataset covers a limited period from March to May 2025.
- Network centrality does not prove expertise, authority, popularity, or real-world influence.
- Sentiment does not directly measure customer satisfaction, product quality, or brand perception.
- The analysis is descriptive and exploratory and does not establish causal relationships.

---

## Ethical Considerations

The dataset contains public Reddit discussion data and usernames.

Responsible use of the analysis requires:

- Avoiding unnecessary identification or profiling of individual users.
- Interpreting centrality as a network property rather than a judgment about a person.
- Avoiding the use of sentiment scores as definitive claims about individual users.
- Considering platform policies and data-access terms for future collection.
- Aggregating or anonymizing user-level findings where appropriate for external business reporting.
- Using community findings as signals for further investigation rather than automated decisions about individuals.

---

## Future Improvements

Potential extensions include:

- Collecting data over a longer time period.
- Creating rolling or monthly community-network comparisons.
- Comparing subreddit-specific networks.
- Measuring how community structure changes over time.
- Adding BERTopic or embedding-based topic modeling.
- Adding semantic clustering using sentence embeddings.
- Comparing additional transformer sentiment models.
- Developing issue/theme detection for product or support research.
- Adding topic trends over time.
- Linking sentiment with network centrality in a more systematic way.
- Creating an interactive dashboard for community, topic, and sentiment exploration.
- Adding automated data collection and scheduled analytics.
- Using stratified sampling or weighting to reduce dominance from high-volume subreddits.
- Separating brand/product-specific discussions from general technical conversations where appropriate.

---

## Business Analysis Takeaway

The central lesson from this project is that online technology communities cannot be understood reliably through a single metric.

Comment volume shows **where activity occurs**.  
Network analysis shows **who interacts with whom**.  
Centrality shows **different structural roles**.  
Community detection shows **how interaction groups form**.  
Sentiment analysis shows **general emotional tone**.  
Topic modeling shows **what people discuss**.

Combining these perspectives creates a more useful analytical framework for stakeholder decision-support than treating engagement, sentiment, or keywords independently.

---

## Author

**Karan Naresh Rathod**

Data Science | Social Network Analysis | Natural Language Processing | Community Analytics
