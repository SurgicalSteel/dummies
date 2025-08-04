### RFC: Comparison of String Similarity Algorithms for Typo Detection

---

#### 1. Introduction  
This document evaluates common string similarity algorithms for typo detection. We compare **Damerau-Levenshtein**, **Jaro-Winkler**, **Levenshtein**, **Cosine Similarity (n-grams)**, and **TF-IDF (n-grams)**. Each algorithm is implemented in Go, with pros/cons analyzed for typo detection in short text (e.g., usernames, search queries).

---

#### 2. Algorithms  
##### 2.1 Damerau-Levenshtein  
**Description**: Extends Levenshtein by including transpositions (swapped adjacent characters). Measures edit operations (insert, delete, substitute, transpose).  
**Implementation**:  
```go
func DamerauLevenshtein(a, b string) int {
    da := make(map[rune]int)
    d := make([][]int, len(a)+2)
    for i := range d {
        d[i] = make([]int, len(b)+2)
    }
    maxDist := len(a) + len(b)
    d[0][0] = maxDist
    for i := 0; i <= len(a); i++ {
        d[i+1][0] = maxDist
        d[i+1][1] = i
    }
    for j := 0; j <= len(b); j++ {
        d[0][j+1] = maxDist
        d[1][j+1] = j
    }
    for i := 1; i <= len(a); i++ {
        db := 0
        for j := 1; j <= len(b); j++ {
            k := da[rune(b[j-1])]
            l := db
            cost := 1
            if a[i-1] == b[j-1] {
                cost = 0
                db = j
            }
            d[i+1][j+1] = min(
                d[i][j]+cost,          // substitution
                d[i+1][j]+1,           // insertion
                d[i][j+1]+1,           // deletion
                d[k][l]+(i-k-1)+1+(j-l-1), // transposition
            )
        }
        da[rune(a[i-1])] = i
    }
    return d[len(a)+1][len(b)+1]
}

func min(nums ...int) int {
    m := nums[0]
    for _, n := range nums {
        if n < m {
            m = n
        }
    }
    return m
}
```

**Pros**:  
- Detects transpositions (common in typos, e.g., "hte" → "the").  
- High accuracy for single-character errors.  
**Cons**:  
- O(mn) time/space complexity (slow for long strings).  
- No normalization; raw edit distance lacks intuitive scoring.  

---

##### 2.2 Jaro-Winkler  
**Description**: Measures common characters and transpositions, then boosts score for matching prefixes.  
**Implementation**:  
```go
func JaroWinkler(s1, s2 string) float64 {
    jaro := Jaro(s1, s2)
    prefix := 0
    for i := 0; i < min(len(s1), len(s2)); i++ {
        if s1[i] != s2[i] {
            break
        }
        prefix++
    }
    prefix = min(4, prefix)
    return jaro + float64(prefix)*0.1*(1-jaro)
}

func Jaro(s1, s2 string) float64 {
    if s1 == s2 {
        return 1.0
    }
    matchDist := max(len(s1), len(s2))/2 - 1
    s1Matches := make([]bool, len(s1))
    s2Matches := make([]bool, len(s2))
    matches := 0
    for i := range s1 {
        start := max(0, i-matchDist)
        end := min(len(s2), i+matchDist+1)
        for j := start; j < end; j++ {
            if !s2Matches[j] && s1[i] == s2[j] {
                s1Matches[i] = true
                s2Matches[j] = true
                matches++
                break
            }
        }
    }
    if matches == 0 {
        return 0.0
    }
    transpositions := 0
    j := 0
    for i := range s1 {
        if !s1Matches[i] {
            continue
        }
        for !s2Matches[j] {
            j++
        }
        if s1[i] != s2[j] {
            transpositions++
        }
        j++
    }
    t := float64(transpositions) / 2
    return (float64(matches)/float64(len(s1)) +
        float64(matches)/float64(len(s2)) +
        (float64(matches)-t)/float64(matches)) / 3
}
```

**Pros**:  
- Fast O(n) complexity.  
- Prefix matching ideal for names/abbreviations (e.g., "Jon" vs "John").  
- Normalized score [0, 1].  
**Cons**:  
- Ignores non-matching suffix errors (e.g., "Smiths" vs "Smith").  
- Weak for middle-character typos (e.g., "Alexander" vs "Alaxander").  

---

##### 2.3 Levenshtein  
**Description**: Minimum single-character edits (insert, delete, substitute) to transform one string to another.  
**Implementation**:  
```go
func Levenshtein(a, b string) int {
    d := make([]int, len(b)+1)
    for j := range d {
        d[j] = j
    }
    for i := 1; i <= len(a); i++ {
        prev := d[0]
        d[0] = i
        for j := 1; j <= len(b); j++ {
            cost := 1
            if a[i-1] == b[j-1] {
                cost = 0
            }
            newVal := min(
                d[j]+1,    // deletion
                d[j-1]+1,  // insertion
                prev+cost,  // substitution
            )
            prev, d[j] = d[j], newVal
        }
    }
    return d[len(b)]
}
```

**Pros**:  
- Simple and widely adopted.  
- Effective for minor typos (e.g., "kitten" → "sitting").  
**Cons**:  
- No transposition support.  
- Unnormalized output; requires length normalization for scores.  

---

##### 2.4 Cosine Similarity (n-grams)  
**Description**: Treats strings as character n-gram vectors and computes cosine of angle between vectors.  
**Implementation**:  
```go
func CosineSimilarity(s1, s2 string, n int) float64 {
    vec1 := toNgramVector(s1, n)
    vec2 := toNgramVector(s2, n)
    dot, mag1, mag2 := 0.0, 0.0, 0.0
    for k, v1 := range vec1 {
        mag1 += float64(v1 * v1)
        if v2, ok := vec2[k]; ok {
            dot += float64(v1 * v2)
        }
    }
    for _, v := range vec2 {
        mag2 += float64(v * v)
    }
    return dot / (math.Sqrt(mag1) * math.Sqrt(mag2))
}

func toNgramVector(s string, n int) map[string]int {
    vec := make(map[string]int)
    for i := 0; i <= len(s)-n; i++ {
        vec[s[i:i+n]]++
    }
    return vec
}
```

**Pros**:  
- Robust to word order changes (e.g., "John Doe" vs "Doe, John").  
- Handles longer typos via n-grams (e.g., "apple" vs "aple" with bigrams).  
**Cons**:  
- Computationally heavy (requires vectorization).  
- Tuning n-gram size critical (small n → false positives).  

---

##### 2.5 TF-IDF (n-grams)  
**Description**: Uses n-gram frequency across a corpus to weight rare n-grams higher.  
**Implementation**:  
```go
type TFIDF struct {
    docs [][]string
    idf  map[string]float64
}

func NewTFIDF(corpus []string, n int) *TFIDF {
    docs := make([][]string, len(corpus))
    for i, s := range corpus {
        docs[i] = toNgrams(s, n)
    }
    idf := make(map[string]float64)
    for _, doc := range docs {
        seen := make(map[string]bool)
        for _, gram := range doc {
            if !seen[gram] {
                seen[gram] = true
                idf[gram]++
            }
        }
    }
    totalDocs := float64(len(docs))
    for gram, count := range idf {
        idf[gram] = math.Log(totalDocs / count)
    }
    return &TFIDF{docs: docs, idf: idf}
}

func (t *TFIDF) Similarity(s1, s2 string) float64 {
    grams1 := toNgrams(s1, 2)
    grams2 := toNgrams(s2, 2)
    vec1, vec2 := make(map[string]float64), make(map[string]float64)
    for _, gram := range grams1 {
        vec1[gram] += t.idf[gram]
    }
    for _, gram := range grams2 {
        vec2[gram] += t.idf[gram]
    }
    dot, mag1, mag2 := 0.0, 0.0, 0.0
    for gram, tfidf1 := range vec1 {
        dot += tfidf1 * vec2[gram]
        mag1 += tfidf1 * tfidf1
    }
    for _, tfidf2 := range vec2 {
        mag2 += tfidf2 * tfidf2
    }
    return dot / (math.Sqrt(mag1) * math.Sqrt(mag2))
}
```

**Pros**:  
- Penalizes common typos (e.g., "teh" vs "the") via IDF.  
- Adaptable to domain-specific corpora.  
**Cons**:  
- Requires precomputed corpus.  
- Overkill for short strings; better for documents.  

---

#### 3. Algorithm Comparison  
| Algorithm               | Typo Detection Strength          | Speed     | Normalized Output | Use Case Example                |
|-------------------------|----------------------------------|-----------|-------------------|---------------------------------|
| Damerau-Levenshtein     | Transpositions (e.g., "caht")    | Slow      | ❌                | Password correction            |
| Jaro-Winkler            | Prefix errors (e.g., "Jhon")     | Fast      | ✅                | Name matching                  |
| Levenshtein             | Single-character edits           | Moderate  | ❌                | Generic spellcheck             |
| Cosine Similarity       | Long typos via n-grams           | Moderate  | ✅                | Product name matching          |
| TF-IDF                  | Contextual importance of errors  | Slow      | ✅                | Document retrieval             |

---

#### 4. Conclusion & Recommendations  
**Best Algorithm**: **Jaro-Winkler** for most typo detection scenarios.  
- **Why**:  
  - Optimized for prefix errors (common in human typing).  
  - Normalized score allows intuitive thresholding (e.g., score > 0.9 = match).  
  - Faster than edit-distance and corpus-based methods.  

**Alternatives**:  
- Use **Damerau-Levenshtein** for applications where transpositions are critical (e.g., data cleaning).  
- Use **Cosine Similarity** for longer strings/phrases with word order variations.  
- Avoid **TF-IDF** for real-time typo detection due to corpus dependency.  

**Final Recommendation**:  
```go
// Typo detection using Jaro-Winkler
func IsTypo(input, target string, threshold float64) bool {
    return JaroWinkler(input, target) >= threshold
}
```