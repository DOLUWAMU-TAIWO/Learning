# 🧠 ZenNest + GCP Technical Meeting Notes

## 🎯 Platform Overview (ZenNest)
- Real estate infrastructure platform for Africa (starting with Nigeria).
- Core Features:
  - Property listings
  - Booking & calendar system
  - User management
  - Payments (Naira, international)
  - Photos & **video uploads** for properties
  - Admin portal
- Growth phase: Startup with foundational infrastructure already in place.

---

## 🎥 Video Intelligence: Key Use Case

### 🎯 Objective
- Let users upload **video listings** of properties.
- Automatically verify that video content is:
  - **Relevant to real estate**
  - **Free of people or faces**
  - **Free from inappropriate content** (e.g., nudity, alcohol, smoking, etc.)
- Use this to **moderate uploads** and ensure platform trust.

### ✅ Questions to Ask
- Does **Google Cloud Video Intelligence API** support:
  - **Scene-level labeling** (e.g., "bedroom", "kitchen", "living room")?
  - Detection of **inappropriate content** (e.g., explicit, violent, etc.)?
  - **Face detection / blurring** tools?
  - Flagging presence of **people, logos, objects**?
  - **Real-time** or near real-time scanning for uploads?
- Can we:
  - **Train custom models** using AutoML or Vertex AI to improve relevance detection?
  - Use **frames + object detection** to verify furniture/room types?
  - Detect **outdoor scenes** vs **indoor**?
  - Use **confidence scores** to auto-approve or flag?

---

## 🛠️ Cloud Services to Discuss

### 📦 Media Storage & Moderation
- Use **Cloud Storage** to store uploaded video files.
- Use **Cloud Functions** or **Cloud Run** to trigger moderation workflows.
- Use **Cloud Vision API** for image-level checks (thumbnails, stills).
- Ask if **Video AI** can export keyframes automatically for quick preview and moderation.

### 🔗 Infrastructure & Backend
- Use **GKE** or **Cloud Run** for microservices.
- Use **Pub/Sub** for triggering events like:
  - New upload → Moderate → Flag or Approve
- Use **Cloud Tasks** for queuing long processing jobs.

---

## 🔍 Analytics & Personalization
- Use **BigQuery** for analyzing:
  - Which listings perform better
  - Upload patterns
  - Flagged content statistics
- Optional: Use **Vertex AI Matching Engine** if content personalization is needed in the future.

---

## 🔒 Compliance & Security
- Ask about:
  - **IAM roles** for secure access to media, storage, and APIs.
  - **KMS (Key Management Service)** for encryption.
  - **Cloud Armor** for security on public endpoints.
  - **GDPR / data retention** policies for uploaded videos.

---

## 💸 Billing & Cost Optimization
- Ask:
  - What’s the **cost model** for Video Intelligence API?
  - Are there **cost-saving tiers** or credits available for startups?
  - How to set **budgets, alerts**, and monitor resource usage?

---

## 🛰️ Advanced Tools You Might Consider Later
- **Vertex AI AutoML Video**: Train custom models for real estate-specific scenes.
- **Dataflow**: Stream processing if large-scale video uploads become common.
- **Looker Studio + BigQuery**: Dashboard for flagged video content and analytics.
- **Cloud Scheduler**: To run nightly scans or re-checks on flagged videos.

---

## ✅ Final Checklist for Meeting
- Clarify real-time vs batch scanning for video.
- Ask if pretrained models can detect real estate environments.
- Ask about scalability and cost of video annotation pipelines.
- Check if Google can offer credits, especially for AI/ML services.