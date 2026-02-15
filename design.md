# VitalPredict AI - Design Document (Hackathon MVP Version)

---

## 1. System Overview

### Description
VitalPredict AI is an intelligent preventive healthcare platform that analyzes blood test parameters and vital signs to predict disease risk using machine learning. The system provides explainable AI insights, personalized recommendations, and a unified health score for users.

### Hackathon Goal
Build a fully working MVP that:
- Predicts cardiovascular disease risk (primary focus)
- Provides explainable AI insights using SHAP
- Generates a unified health score (0–100)
- Delivers personalized preventive recommendations
- Demonstrates scalability roadmap for future expansion

---

## 2. MVP Scope (What Will Be Built)

### Included in Hackathon Version
- User Registration & Login (JWT-based)
- Health Data Entry Form
- Cardiovascular Disease Risk Model (XGBoost)
- Risk Prediction API
- Unified Health Score Calculation
- SHAP-based Explainability
- Rule-based Recommendation Engine
- Basic Dashboard (Risk gauge + charts)
- Dockerized Deployment (Single EC2 instance)

### Deferred to Future Phases
- Multi-disease prediction (Diabetes, Stroke, Kidney)
- Wearable integrations
- AI Health Twin forecasting
- EHR integration
- Advanced DevOps scaling (ECS/EKS)
- Multi-replica databases
- Terraform-based infrastructure

---

## 3. High-Level Architecture (MVP)

Client (React Frontend)
↓
FastAPI Backend (Dockerized)
↓
ML Model (XGBoost + SHAP)
↓
PostgreSQL Database
↓
Redis Cache (optional for MVP)


---

## 4. Technology Stack

### Frontend
- React 18 + TypeScript
- Tailwind CSS
- Axios (API communication)
- Recharts / Chart.js (visualizations)

### Backend
- FastAPI (Python 3.10+)
- SQLAlchemy
- Pydantic
- JWT Authentication
- Uvicorn

### ML/AI
- XGBoost (CVD model)
- Scikit-learn (preprocessing)
- SHAP (Explainability)
- Pandas, NumPy

### Database
- PostgreSQL (Primary storage)
- Redis (Optional caching)

### Deployment (MVP)
- Docker
- AWS EC2 (Single instance)
- Nginx (Reverse proxy)

---

## 5. Data Flow

### Prediction Flow

1. User logs in (JWT authentication)
2. User submits health data
3. Backend validates input
4. Data stored in PostgreSQL
5. Preprocessing pipeline applied:
   - Feature engineering
   - Scaling
   - Derived metrics
6. Model loaded in memory
7. Risk probability calculated
8. SHAP values generated
9. Unified health score computed
10. Recommendations generated
11. Results returned to frontend

---

## 6. ML Model Design (MVP)

### Target Disease
Cardiovascular Disease (Primary)

### Input Features

#### Clinical Biomarkers
- Total Cholesterol
- LDL
- HDL
- Triglycerides
- Systolic BP
- Diastolic BP
- Fasting Glucose
- BMI

#### Demographics
- Age
- Gender
- Smoking Status
- Family History

#### Derived Features
- Cholesterol Ratio (Total/HDL)
- Pulse Pressure
- Mean Arterial Pressure

### Model Details
- Algorithm: XGBoost Classifier
- Train/Test Split: 80/20
- Cross-validation: 5-fold
- Target Accuracy: >85%
- AUC-ROC: >0.85
- Inference Time: <50ms

---

## 7. Risk Scoring Logic

### Risk Classification
- Low: 0–20% → Green
- Moderate: 21–50% → Yellow
- High: 51–100% → Red

### Unified Health Score

### Health Score Interpretation
- 80–100: Excellent
- 60–79: Good
- 40–59: Fair
- 20–39: Poor
- 0–19: Critical

---

## 8. Recommendation Engine (MVP)

Rule-based system mapping risk factors to interventions.

| Risk Factor | Recommendation |
|------------|---------------|
| High LDL | Reduce saturated fat intake |
| High BP | Reduce sodium, daily walking |
| High BMI | Calorie deficit plan |
| Smoking | Smoking cessation program |

Each recommendation includes:
- Category (Diet / Exercise / Lifestyle)
- Priority Level
- Expected Impact
- Suggested Timeline

---

## 9. Security & Privacy (MVP Level)

- JWT Authentication (15 min access token)
- Password hashing (bcrypt)
- Role-based access (User / Admin)
- TLS encryption (HTTPS)
- Input validation (Pydantic)
- SQL injection prevention (ORM)

---

## 10. Database Schema (Simplified)

### users
- id
- email
- password_hash
- role
- created_at

### health_data
- id
- user_id
- cholesterol
- ldl
- hdl
- systolic_bp
- diastolic_bp
- glucose
- bmi
- smoking
- family_history
- created_at

### predictions
- id
- user_id
- health_data_id
- cvd_risk
- health_score
- created_at

### explanations
- id
- prediction_id
- feature_name
- shap_value
- contribution_pct
- rank

---

## 11. Unique Innovation Features (Demo Focus)

### 1. What-If Simulation
Users can modify input values and instantly see how risk changes without saving data.

### 2. Explainable AI
SHAP-based visual explanation showing top contributing factors.

### 3. Personalized Health Score
Unified 0–100 score for easy understanding.

### 4. Visual Dashboard
- Risk Gauge
- Feature Importance Chart
- Recommendation Cards
- Risk History Graph

---

## 12. Deployment Plan (Hackathon)

- Single EC2 instance
- Dockerized backend + frontend
- PostgreSQL running via Docker
- Nginx reverse proxy
- GitHub Actions (basic CI/CD optional)

---

## 13. Future Scalability Roadmap

After hackathon validation:
- Add Diabetes, Stroke, Kidney models
- Deploy on ECS/EKS
- Add wearable integrations
- Implement AI Health Twin forecasting
- Multi-language support
- Federated learning for privacy

---

## 14. Success Criteria

### Technical
- API response <500ms
- Model accuracy >85%
- No runtime crashes
- Clean UI demo

### Impact
- Clear disease risk insights
- Actionable recommendations
- Strong preventive healthcare positioning

---

**End of Document**
