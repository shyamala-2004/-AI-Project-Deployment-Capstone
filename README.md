# -AI-Project-Deployment-Capstone
# Amazon Product Rating Predictor

An end-to-end Machine Learning project: data preprocessing → model training → evaluation → serialization (Joblib) → deployment as a **Flask** prediction API with a small web UI.

**Problem:** Given a product's price, discount, number of ratings, category and listing text length, predict whether it will be **top rated (rating ≥ 4.2)**.

**Dataset:** `data/amazon.csv` – 1,465 Amazon India product listings (1,350 unique products after cleaning).

## Results (held-out 20 % test set)

| Metric | Value |
| Accuracy | 72.6 % (majority-class baseline: 53.7 %) |
| F1 (top rated) | 0.689 |
| ROC-AUC | 0.798 |

Model comparison (5-fold CV accuracy on training data): Logistic Regression 0.665 · Gradient Boosting 0.684 · **Random Forest 0.744** (selected, tuned with GridSearchCV).

## Project structure

```
amazon-rating-predictor/
├── data/amazon.csv            # raw dataset
├── src/
│   ├── config.py              # paths, features, constants
│   ├── preprocess.py          # cleaning + feature engineering
│   └── train.py               # model comparison, tuning, evaluation, joblib.dump
├── models/
│   ├── rating_model.joblib    # serialized preprocessing+model pipeline
│   └── metrics.json           # evaluation results
├── app/
│   ├── app.py                 # Flask API (/, /health, /predict)
│   └── templates/index.html   # web form
├── tests/                     # pytest unit + API tests
├── reports/                   # generated charts
├── docs/                      # project report (.docx) and presentation (.pptx)
├── requirements.txt
├── Dockerfile
├── Procfile                   # for Render / Heroku
└── README.md
```

## Setup & run

```bash
python -m venv venv && source venv/bin/activate     # Windows: venv\Scripts\activate
pip install -r requirements.txt

python -m src.train          # trains and saves models/rating_model.joblib
python -m pytest -q          # run tests
python -m app.app            # starts http://localhost:5000
```

## API

`GET /health` → `{"status": "ok", "model": "RandomForest"}`

`POST /predict` (JSON):

```bash
curl -X POST http://localhost:5000/predict -H "Content-Type: application/json" -d '{
  "discounted_price": 399,
  "actual_price": 1099,
  "rating_count": 24269,
  "category": "Computers&Accessories|Accessories&Peripherals|Cables&Accessories|Cables|USBCables",
  "product_name": "Wayona Nylon Braided USB to Lightning Cable",
  "about_product": "Durable nylon braided design, fast charge and data sync"
}'
```

Response: `{"prediction": "Top rated", "top_rated_probability": 0.8933}`

Required: `discounted_price`, `actual_price`, `rating_count`, `category`. Optional: `product_name`, `about_product`. Invalid input returns HTTP 400 with an `error` message.

## Deployment

* **Docker:** `docker build -t rating-api . && docker run -p 5000:5000 rating-api`
* **Render / Heroku:** push to GitHub, create a Web Service, build command `pip install -r requirements.txt && python -m src.train`, start command from `Procfile`.

## Limitations & future work

Ratings depend heavily on product quality that is not in this table; adding review-text features (TF-IDF / sentiment) and more data should improve accuracy. The model is trained on Amazon India listings and may not transfer to other marketplaces.

