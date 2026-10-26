# SEO Internal Linking Keyword Visualizer

Python tool to crawl and visualize contextual internal links for URLs containing a specific keyword.

## What it does

- Crawls internal links.
- Filters URLs containing a chosen keyword, such as `avoriaz`.
- Excludes menu, header and navigation links.
- Generates a visual internal linking graph.
- Shows the URL of each node.
- Helps identify internal linking gaps and content clusters.

## Requirements

```bash
pip install requests beautifulsoup4 networkx matplotlib
```

## Usage

Edit these values in the script:

```python
base_url = "https://www.example.com"
```

and change:

```python
r"avoriaz"
```

to the keyword you want to analyse.

Then run:

```bash
python internal-linking-keyword.py
```
