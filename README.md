
# Movie Memory

**AI 301: Natural Language Processing and Conversational AI — Fall 2026**

**Group 1:** Arianna Rogers, Drew Woodard, Dylan Welby, and Lauren Willis

## Project Overview

Movie Memory is a system designed to identify movies from vague plot descriptions. The project compares traditional keyword search with semantic search and uses retrieval-augmented generation (RAG) to identify likely movie titles and provide explanations.

Our main research question is whether semantic search and RAG can identify movies from vague descriptions more effectively than traditional keyword search.

## Dataset

We use the Wiki Movie Plots with Summaries dataset, containing 34,886 movie records.

Each record includes information such as the movie title, release year, plot summary, and Wikipedia source.

- [Original dataset on Hugging Face](https://huggingface.co/datasets/vishnupriyavr/wiki-movie-plots-with-summaries)
- [Dataset CSV on Google Drive](https://drive.google.com/file/d/1GNBx6j23REDS9Cw94sx1sYx1zP663Juc/view?usp=sharing)
- License: CC BY-SA 4.0

The notebook automatically downloads the dataset from Google Drive.

## Methods and Models

Our project implements four approaches:

1. **BM25 Keyword Search:** Ranks movies based on keyword matches between the user's description and movie plot summaries.
2. **Semantic Search with FAISS:** Uses `thenlper/gte-small` to generate embeddings and FAISS to retrieve similar movie plots.
3. **Qwen-Only Baseline:** Uses `Qwen/Qwen2.5-1.5B-Instruct` to identify movies without retrieved plot information.
4. **Retrieval-Augmented Generation:** Retrieves relevant movie plots with FAISS and provides them to Qwen to identify a movie and explain the selection.

We also compare two RAG configurations: providing Qwen with the top three versus the top five retrieved movie plots.

## How to Run the Project

### 1. Open the Notebook

Open `AI301_Group1_Final.ipynb` in Google Colab.

### 2. Enable GPU Acceleration

In Google Colab, select:

**Runtime → Change runtime type → T4 GPU**, if available.

GPU acceleration is recommended for running Qwen and generating embeddings.

### 3. Run the Notebook

Select **Runtime → Run all**.

The notebook installs the required libraries, downloads the dataset, prepares the test cases, loads the models, generates embeddings, builds the FAISS index, and evaluates each approach.

An internet connection is required to download the dataset, test cases, and pretrained models.

The first run may take some time because the notebook processes 34,886 movie plot summaries and generates answers using Qwen.

No API keys are required.

### 4. Review the Results

The notebook displays results for:

- BM25 keyword search
- Qwen without retrieval
- FAISS semantic search
- RAG with five retrieved plots
- RAG with three retrieved plots

The notebook also exports individual evaluation results, a consolidated results table, and the final test set as CSV files.

## Dependencies

The project was tested in Google Colab using the following package versions:

| Package | Version |
|---|---|
| pandas | 2.2.3 |
| numpy | 2.4.6 |
| torch | 2.14.1+cu130 |
| transformers | 5.22.1 |
| sentence-transformers | 5.7.0 |
| faiss-cpu | 1.16.0 |
| rank-bm25 | 0.2.2 |
| gdown | 5.2.2 |

The notebook installs the additional required packages in its setup cells and displays the installed package versions at the end.

These versions document the tested environment. The installation cells do not explicitly pin every package to these versions.

## Model Settings

### Retrieval Settings

| Setting | Value |
|---|---|
| Embedding model | thenlper/gte-small |
| Embedding dimensions | 384 |
| Embedding normalization | Enabled |
| FAISS index | IndexFlatIP |
| Retrieval evaluation | Top 5 |
| RAG configurations | Top 3 and Top 5 |

### Generation Settings

| Setting | Qwen-Only | RAG |
|---|---|---|
| Language model | Qwen2.5-1.5B-Instruct | Qwen2.5-1.5B-Instruct |
| Maximum new tokens | 100 | 150 |
| Random sampling | Disabled | Disabled |
| Retrieved plots | None | 3 or 5 |
| Candidate-title validation | No | Yes |

Random sampling is disabled using `do_sample=False`. No explicit random seed is set. Results may vary across different hardware and software environments.

## Test Cases

We created 20 vague movie descriptions to evaluate the system.

- 18 descriptions have identifiable movies in the dataset.
- 2 descriptions are intentionally unanswerable.
- Answerable cases include expected movie titles, release years, Record IDs, and sources.

The test cases are available in `movie_memory_20_test_cases.csv`.

The notebook also uses `movie_memory_additional_15_test_cases.csv` from this repository and combines those records with the five original examples. Six cases are then revised to improve decade and genre diversity.

## Evaluation Metrics

- **Hit Rate@5:** The percentage of answerable test cases where the expected movie appears among the five retrieved results.
- **Title Accuracy:** The percentage of answerable test cases where the model identifies the expected movie title.
- **Abstention Accuracy:** The percentage of intentionally unanswerable test cases where the model correctly declines to identify a movie.

## Evaluation Results

| Method | Metric | Result |
|---|---|---|
| BM25 Keyword Search | Hit Rate@5 | 16.67% (3/18) |
| FAISS Semantic Search | Hit Rate@5 | 38.89% (7/18) |
| Qwen-Only Baseline | Title Accuracy | 16.67% (3/18) |
| RAG — Top 5 Plots | Title Accuracy | 33.33% (6/18) |
| RAG — Top 3 Plots | Title Accuracy | 33.33% (6/18) |

BM25 and semantic search are evaluated using Hit Rate@5, while Qwen and RAG are evaluated using movie-title accuracy. These metrics measure different aspects of system performance.

### Abstention Results

| Method | Correct Abstentions | Abstention Accuracy |
|---|---|---|
| Qwen-Only Baseline | 1/2 | 50.00% |
| RAG — Top 5 Plots | 0/2 | 0.00% |
| RAG — Top 3 Plots | 0/2 | 0.00% |

## Design Experiment

We compared two versions of the RAG system while keeping the embedding model, language model, prompt structure, and test descriptions the same.

The only experimental change was the number of retrieved movie plots provided to Qwen.

Both configurations correctly identified 6 of 18 answerable movies, achieving 33.33% accuracy.

This suggests that reducing the number of retrieved plots from five to three did not improve overall accuracy on our test set.

## Limitations

Our system has several limitations:

- Semantic search sometimes retrieves movies that do not match the intended description.
- Qwen can select an incorrect movie even when the correct movie appears among the retrieved candidates.
- Neither RAG configuration correctly rejected the two unanswerable test descriptions.
- The evaluation uses only 20 test cases, so the results may not represent performance on a larger collection of descriptions.
- Generated explanations may contain details that are not fully supported by the retrieved movie plots.
- Movies with identical titles but different release years may complicate title-based evaluation.
- The Qwen-only and RAG configurations use different prompts and maximum token limits.

## Repository Contents

| File | Description |
|---|---|
| AI301_Group1_Final.ipynb | Complete project notebook |
| README.md | Project documentation and setup instructions |
| movie_memory_additional_15_test_cases.csv | Additional test cases used by the notebook |
| movie_memory_20_test_cases.csv | Final test set and answer key |
| movie_memory_final_results.csv | Consolidated evaluation results |
| bm25_results.csv | Individual BM25 evaluation results |
| semantic_results.csv | Individual FAISS evaluation results |
| qwen_results.csv | Individual Qwen-only evaluation results |
| rag_top5_results.csv | Individual top-5 RAG evaluation results |
| rag_top3_results.csv | Individual top-3 RAG evaluation results |

## Repository

GitHub: https://github.com/aroger78/Movie-Memory

## Team Members

- Arianna Rogers
- Drew Woodard
- Dylan Welby
- Lauren Willis

Individual project contributions are documented in the notebook.
