# korean text tokenization using Python within R
This repository demonstrates how to perform Korean text tokenization using the `kiwipiepy` library through R's `reticulate` package. 
The example shows how to extract nouns from Korean text data and process them in a dataframe. 

As the traditional KoNLP package is no longer available on CRAN and installing rJava often causes compatibility issues, this approach provides a reliable alternative for Korean text analysis. By leveraging Python's kiwipiepy through R's reticulate, users can easily perform text tokenization without dealing with Java dependencies or compatibility problems.

The example shows how to extract nouns from Korean text data and process them in a dataframe, offering a straightforward solution for Korean natural language processing in R. The code adds tokenized results as a new column in the original R data frame, allowing users to seamlessly utilize R's powerful data manipulation and analysis tools (like tidyverse) with the tokenized outcomes.


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

Output will look like this:

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

## Notes

- The code extracts only nouns (NNG: common nouns, NNP: proper nouns)
- Progress tracking is included through tqdm
- The original dataframe structure is preserved

## Language Tags in Kiwi
You can modify the tag filtering in `process_text()` to extract different parts of speech as needed.

Common tags used in this example:
- NNG: General Noun (일반 명사)  - e.g., 책, 학교
- NNP: Proper Noun (고유 명사)  - e.g., 서울, 철수
- NNB: 의존 명사 (Dependent Noun) - e.g., 것, 수
- NR: 수사 (Numeral) - e.g., 하나, 둘
- NP: 대명사 (Pronoun) - e.g., 나, 너
- VV: 동사 (Verb) - e.g., 먹다, 가다
- VA: 형용사 (Adjective) - e.g., 예쁘다, 크다
- VX: 보조 용언 (Auxiliary Verb) - e.g., ~하다, ~되다
- VCP: 긍정 지정사 (Copula) - 이다
- VCN: 부정 지정사 (Negative Copula) - 아니다
- MM: 관형사 (Determinative) - e.g., 이, 그, 저
- MAG: 일반 부사 (General Adverb) - e.g., 매우, 빨리
- MAJ: 접속 부사 (Conjunctive Adverb) - e.g., 그러나, 하지만


## Contributing
Feel free to submit issues and enhancement requests!

## License
This project is licensed under the MIT License - see the LICENSE file for details.
