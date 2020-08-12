# Final Demo Outline

## Frontend Demo (CRUD)

### Mobile Frontend

1. Login
2. Calendar
3. Popular Courses
4. Search/Add/Delete Courses
5. Add/Get/Remove Remarks
6. GPA Visualization
7. Professor Rating
8. Professor Comments
9. Course Difficulty/Workload

### Chatbot

1. Link Account
2. Get Enrollment
3. Add Room

## Real Data

### Storage and Query

1. Database Walk-through

   > Having **real data in the system** (5%)  If you crawl data, you must have at least 100 records in the DB. We have 300K.

2. Elasticsearch

   + Why?
     + Prof names doesn't match.
     + ~~Recommender~~. Don't have time.
   + Detail
     + Whitespace tokenizer
     + Edge-Ngram filter

### Data-fetching [Advanced Funtion]

1. Course Crawler
2. Ratemyprofessor Crawler

### "Interesting Queries"

1. Fuzzy search for courses
2. Get Remark

## Backend Walk-through

### Backend Architecture

+ VPC
+ API Gateway
+ Elastic-computing
+ Database "cluster" (Reader + Writer)
+ Elasticsearch "cluster"Professor Rating

### API Walk-through

+ Code
+ Endpoint visualize

### Course Difficulty Metrics[Advanced Function]

+ Data
+ Formula
+ Challenge