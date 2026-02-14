# VitalPredict AI - Design Document

---

## 1. System Overview

**Description:**
VitalPredict AI is an intelligent preventive healthcare platform that analyzes blood test parameters and vital signs to predict multi-disease risks using machine learning. The system provides explainable AI insights, personalized recommendations, and risk tracking for individuals and healthcare providers.

**Goals:**
- Predict risk for cardiovascular disease, diabetes, stroke, and kidney disease
- Provide explainable AI showing contributing factors
- Generate unified health score (0-100)
- Deliver personalized preventive recommendations
- Support both patients and doctors
- Scale to 1M+ users with <500ms response time

**Target Users:**
- Individual users seeking preventive health insights
- Healthcare providers monitoring patient populations
- Health coaches and wellness professionals

---

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Web App     │  │ Mobile App   │  │   Doctor     │      │
│  │  (React)     │  │  (Future)    │  │  Dashboard   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                       │
│         Authentication | Rate Limiting | CORS                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer (FastAPI)               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   User   │  │  Health  │  │Prediction│  │  Recom-  │   │
│  │ Service  │  │  Service │  │ Service  │  │ mendation│   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        ML Layer                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   CVD    │  │ Diabetes │  │  Stroke  │  │  Kidney  │   │
│  │  Model   │  │  Model   │  │  Model   │  │  Model   │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                    SHAP Explainer                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                        Data Layer                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │PostgreSQL│  │TimescaleDB│ │  Redis   │  │    S3    │   │
│  │  (Main)  │  │ (History) │  │ (Cache)  │  │ (Models) │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Technology Stack

**Frontend:**
- React 18 + TypeScript
- Redux Toolkit (state management)
- Material-UI or Tailwind CSS
- Recharts / Chart.js (visualizations)
- Axios (API client)

**Backend:**
- FastAPI (Python 3.10+)
- SQLAlchemy (ORM)
- Pydantic (validation)
- JWT authentication
- Uvicorn (ASGI server)

**ML/AI:**
- XGBoost (primary model)
- Scikit-learn (preprocessing)
- SHAP (explainability)
- Pandas, NumPy (data processing)
- MLflow (model registry)

**Database:**
- PostgreSQL 15 (primary database)
- TimescaleDB (time-series data)
- Redis 7 (caching, sessions)
- AWS S3 (model artifacts, backups)

**Infrastructure:**
- Docker (containerization)
- AWS ECS/EKS (orchestration)
- CloudFront (CDN)
- Terraform (IaC)

**DevOps:**
- GitHub Actions (CI/CD)
- CloudWatch (monitoring)
- Sentry (error tracking)
- Prometheus + Grafana (metrics)

---

## 4. Data Flow

**User Input → Prediction Flow:**

```
1. User Authentication
   → JWT token validation
   → Role-based access check (Patient/Doctor)

2. Health Data Submission
   → Frontend validation (Yup schema)
   → API request with encrypted payload
   → Backend validation (Pydantic models)
   → Physiological range checks
   → Store in PostgreSQL with timestamp

3. Risk Prediction
   → Load user's latest health data
   → Preprocessing pipeline:
      • Feature engineering (ratios, derived metrics)
      • Scaling/normalization
      • Encoding categorical variables
   → Load ML models from cache
   → Parallel prediction execution (CVD, Diabetes, Stroke, Kidney)
   → Calculate unified health score
   → Generate SHAP explanations
   → Generate personalized recommendations
   → Store results in database
   → Cache in Redis (5-min TTL)
   → Return JSON response

4. Dashboard Rendering
   → Parse prediction data
   → Render risk score gauge
   → Display disease risk cards
   → Show SHAP feature importance
   → List recommendations
   → Enable PDF/CSV export
```

**API Validation:**
- Age: 18-120 years
- Cholesterol: 100-500 mg/dL
- Blood Pressure: 80-200 / 50-130 mmHg
- Glucose: 50-400 mg/dL
- HbA1c: 4.0-15.0%
- BMI: 10-60 kg/m²

---

## 5. ML Model Design

**Input Features (21 total):**

*Clinical Biomarkers (9):*
- Total Cholesterol, LDL, HDL, Triglycerides
- Systolic BP, Diastolic BP
- Fasting Glucose, HbA1c, Creatinine

*Demographics & Lifestyle (6):*
- Age, Gender, BMI
- Smoking Status, Physical Activity, Family History

*Derived Features (6):*
- Cholesterol Ratio (Total/HDL)
- Atherogenic Index (LDL/HDL)
- Pulse Pressure (Systolic - Diastolic)
- eGFR (kidney function)
- Non-HDL Cholesterol
- Mean Arterial Pressure

**Model Architecture:**
- **Algorithm**: XGBoost Classifier (separate model per disease)
- **Why XGBoost**: High accuracy, fast training, built-in feature importance, handles missing data
- **Training**: 80-20 split, 5-fold cross-validation, SMOTE for class imbalance
- **Hyperparameters**: max_depth=6, learning_rate=0.1, n_estimators=100

**Output:**
- Disease risk probabilities (0-100%)
- Risk classification (Low <20%, Moderate 20-50%, High >50%)
- Unified health score (0-100)
- Top 5 contributing factors per disease
- SHAP values for explainability

**Model Performance Targets:**
- Accuracy: >85% for CVD, Diabetes, Stroke
- Accuracy: >80% for Kidney Disease
- AUC-ROC: >0.85
- Inference time: <50ms per prediction

**Explainability (SHAP):**
- Calculate SHAP values for each prediction
- Identify top contributing factors
- Quantify feature impact as percentage
- Generate human-readable explanations

---

## 6. Risk Scoring Logic

**Unified Health Score Formula:**
```
Health Score = 100 - Weighted Risk Score

Weighted Risk = (0.40 × CVD_Risk) + (0.30 × Diabetes_Risk) 
                + (0.20 × Stroke_Risk) + (0.10 × Kidney_Risk)
```

**Risk Classification:**
- **Low**: 0-20% probability → Green 🟢
- **Moderate**: 21-50% probability → Yellow 🟡
- **High**: 51-100% probability → Red 🔴

**Health Score Interpretation:**
- 80-100: Excellent (very low risk)
- 60-79: Good (low to moderate risk)
- 40-59: Fair (moderate risk)
- 20-39: Poor (high risk)
- 0-19: Critical (very high risk)

**Recommendation Engine:**
- Rule-based system mapping risk factors to interventions
- Priority: Critical > High > Medium > Low
- Categories: Diet, Exercise, Lifestyle, Medical
- Expected impact and timeline included
- Personalized based on user's specific risk factors

---

## 7. Security & Privacy

**Authentication:**
- JWT tokens (access: 15-min, refresh: 7-day)
- Password hashing with bcrypt
- Role-based access control (Patient, Doctor, Admin)
- Multi-factor authentication (optional)

**Data Encryption:**
- At rest: AES-256 encryption for sensitive fields
- In transit: TLS 1.3 for all communications
- Database: PostgreSQL TDE enabled
- Keys: AWS KMS with 90-day rotation

**HIPAA-Inspired Compliance:**
- Minimum necessary access principle
- Comprehensive audit logging
- 15-minute session timeout
- Data anonymization for analytics
- 30-day soft delete for account deletion
- 7-year audit log retention

**Security Controls:**
- Rate limiting (100 req/min per user)
- IP whitelisting for admin access
- SQL injection prevention (parameterized queries)
- XSS protection (input sanitization)
- CSRF tokens for state-changing operations

---

## 8. Scalability Plan

**Horizontal Scaling:**
- Stateless API servers (no session state)
- Auto-scaling: 2-20 ECS instances based on CPU/memory
- Target: 10,000 requests/second

**Database Optimization:**
- Read replicas (3) for analytics queries
- Connection pooling (PgBouncer): 100 connections
- Indexing on user_id, created_at, risk levels
- Time-based partitioning for historical data

**Caching Strategy:**
- Prediction results: 5-minute TTL
- User profiles: 1-hour TTL
- Model artifacts: No expiry
- Session data: 15-minute TTL

**Load Balancing:**
- Application Load Balancer (ALB)
- Health checks every 30 seconds
- Round-robin distribution
- Sticky sessions disabled

**Model Serving:**
- Load models at startup (not per request)
- Async/await for parallel predictions
- Model quantization for faster inference
- Batch predictions when possible

**Performance Targets:**
- API response time: <500ms (p95)
- Model inference: <50ms
- Database query: <100ms
- Cache hit rate: >80%

---

## 9. Unique Innovation Features

**1. Real-Time "What-If" Simulation**
- Interactive sliders to modify health parameters
- See instant risk changes without saving data
- Side-by-side comparison (current vs. simulated)
- Goal-based reverse calculation

**2. AI Health Twin (5-Year Forecast)**
- Predict health trajectory based on current trends
- Forecast risk scores for 1, 2, 3, 4, 5 years ahead
- Show confidence intervals
- Compare "current path" vs. "recommended path"

**3. Personalized AI Plans**
- Custom meal plans based on risk factors
- Exercise routines tailored to fitness level
- Weekly schedules and shopping lists
- Evidence-based interventions

**4. Smart Wearable Integration**
- Sync data from Apple Watch, Fitbit, Garmin
- Continuous heart rate and activity monitoring
- Automatic risk updates on significant changes
- Anomaly detection and alerts

**5. Early Warning System**
- Proactive alerts for risk level changes
- Notifications for health score drops >10 points
- Critical value alerts (BP >160/100, glucose >200)
- Trend deterioration warnings

---

## 10. Limitations & Future Scope

**Current Limitations:**
- Requires manual data entry (no EHR integration yet)
- Limited to 4 diseases (CVD, Diabetes, Stroke, Kidney)
- No genetic risk factors included
- English language only
- Web-only (no mobile app)

**Future Enhancements:**

*Phase 2 (3-6 months):*
- Mobile apps (iOS, Android)
- EHR integration (HL7 FHIR)
- Additional diseases (cancer, liver, osteoporosis)
- Multi-language support
- Telemedicine integration

*Phase 3 (6-12 months):*
- Real-time biosensor integration (CGM, continuous BP)
- AI health chatbot (GPT-4 powered)
- Genetic risk integration (23andMe, AncestryDNA)
- Social features (challenges, communities)
- Insurance integration

*Research & Development:*
- Deep learning models (LSTM for time-series)
- Federated learning (privacy-preserving)
- Causal inference models
- Counterfactual explanations
- Natural language report generation

---

## Database Schema (Key Tables)

**users**: id, email, password_hash, role, first_name, last_name, dob, gender, created_at

**health_data**: id, user_id, cholesterol, ldl, hdl, triglycerides, bp, glucose, hba1c, creatinine, bmi, smoking, activity, family_history, created_at

**predictions**: id, user_id, health_data_id, cvd_risk, diabetes_risk, stroke_risk, kidney_risk, health_score, model_version, created_at

**explanations**: id, prediction_id, disease_type, feature_name, feature_value, shap_value, contribution_pct, rank

**recommendations**: id, prediction_id, category, priority, title, description, expected_impact, timeline

**audit_logs**: id, user_id, action, resource_type, resource_id, ip_address, details, created_at

---

## API Endpoints (Core)

**Auth**: POST /auth/register, /auth/login, /auth/refresh, /auth/logout

**Users**: GET /users/me, PUT /users/me, DELETE /users/me

**Health Data**: POST /health-data, GET /health-data, GET /health-data/latest

**Predictions**: POST /predict, GET /predictions, GET /predictions/latest

**Explanations**: GET /explanations/{prediction_id}

**Recommendations**: GET /recommendations, GET /recommendations/{prediction_id}

**Analytics**: GET /analytics/trends, /analytics/risk-history, /analytics/trajectory

**Doctor**: GET /doctor/patients, GET /doctor/patients/{id}, GET /doctor/analytics

**Export**: GET /export/pdf, /export/csv, /export/json

---

## Deployment Architecture (AWS)

**Compute**: ECS Fargate (2-20 instances, auto-scaled)

**Database**: RDS PostgreSQL Multi-AZ, 3 read replicas

**Cache**: ElastiCache Redis cluster (3 nodes)

**Storage**: S3 (models, exports, backups)

**CDN**: CloudFront (global distribution)

**Networking**: VPC, public/private subnets, security groups

**Monitoring**: CloudWatch (logs, metrics, alarms), X-Ray (tracing)

**CI/CD**: GitHub Actions → Docker build → ECR → ECS deployment

**Cost Estimate**: $665-$1,465/month (production)

---

## Success Metrics

**Technical KPIs:**
- API response time: <500ms (p95)
- System uptime: >99.9%
- Model accuracy: >85%
- Zero data breaches

**Business KPIs:**
- 10,000 users in 3 months
- 60% monthly retention
- 3+ predictions per user/month
- 100+ healthcare providers

**Health Impact:**
- 10% average health score improvement
- 50% users adopt recommendations
- 30% high-risk users consult doctors

---

**End of Design Document**

