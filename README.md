# koreantexttokenization
This repository demonstrates how to perform Korean text tokenization using the `kiwipiepy` library through R's `reticulate` package. 
The example shows how to extract nouns from Korean text data and process them in a dataframe.

## Prerequisites

- R with tidyverse
- Python
- reticulate R package
- kiwipiepy Python package
- tqdm Python package (optional, for progress bars)

## Installation

```r
# In R
install.packages(c("tidyverse", "reticulate"))
```

## Example Code

First, let's create a sample dataset with Korean text:

```r
library(tidyverse)
library(reticulate)

# Create sample data
sample_df <- tibble(
  id = 1:5,
  date = as.Date("2024-01-01") + 0:4,
  comment = c(
    "오늘 날씨가 정말 좋았습니다",
    "서울 시청 근처 맛집을 찾고 있어요",
    "인공지능 기술이 발전하고 있습니다",
    "한국 문화가 세계적으로 인기가 많아요",
    NA  # Including NA to demonstrate null handling
  )
)

# Install required Python packages
py_install("kiwipiepy", pip = TRUE)
py_install("tqdm", pip = TRUE)
```

Now, let's set up the Python functions for text processing:

```r
# Define Python functions through reticulate
py_run_string("
from kiwipiepy import Kiwi
import pandas as pd
from tqdm import tqdm

def process_text(text):
    try:
        if pd.isna(text):
            return ''
            
        kiwi = Kiwi()
        result = kiwi.analyze(str(text))
        
        # Extract only nouns
        nouns = []
        for token in result[0][0]:
            if token.tag in ['NNG', 'NNP']:  # Common and proper nouns
                nouns.append(token.form)
                
        return ' '.join(nouns)
    except Exception as e:
        print(f'Error: {e}')
        return ''

def process_dataframe(texts):
    # Show progress with tqdm
    return [process_text(text) for text in tqdm(texts, desc='Tokenizing')]
")
```

Process the sample data:

```r
# Test the functions
tokenized_texts <- py$process_dataframe(sample_df$comment)

# Add tokenized text back to the dataframe
result_df <- sample_df %>%
  ungroup() %>%
  mutate(tokenized = tokenized_texts)

# View results
print(result_df)
```

Expected output will look something like this:

```r
# A tibble: 5 × 4
     id date       comment                                tokenized
  <int> <date>     <chr>                                 <chr>    
1     1 2024-01-01 오늘 날씨가 정말 좋았습니다             오늘 날씨
2     2 2024-01-02 서울 시청 근처 맛집을 찾고 있어요        서울 시청 맛집
3     3 2024-01-03 인공지능 기술이 발전하고 있습니다        인공지능 기술
4     4 2024-01-04 한국 문화가 세계적으로 인기가 많아요     한국 문화 세계
5     5 2024-01-05 NA                                    ""       
```

## Explanation

This code does the following:

1. Creates a sample dataframe with Korean text data
2. Sets up Python environment through reticulate
3. Defines two Python functions:
   - `process_text()`: Processes individual text entries
   - `process_dataframe()`: Processes multiple texts with progress tracking
4. Extracts nouns using Kiwi analyzer
5. Handles null values and exceptions
6. Returns tokenized results as space-separated strings

## Notes

- The code extracts only nouns (NNG: common nouns, NNP: proper nouns)
- Empty or NA values are handled gracefully
- Progress tracking is included through tqdm
- The original dataframe structure is preserved

## Language Tags in Kiwi

Common tags used in this example:
- NNG: General Noun (일반 명사)
- NNP: Proper Noun (고유 명사)

You can modify the tag filtering in `process_text()` to extract different parts of speech as needed.

## Contributing

Feel free to submit issues and enhancement requests!

## License

This project is licensed under the MIT License - see the LICENSE file for details.
