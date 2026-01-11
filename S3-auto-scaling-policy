### 📈 Auto Scaling Policy

- **Metric:** Average CPU Utilization  
- **Target:** 60%  
- **Scale Out:** +1 instance when CPU > 60% for 2 minutes  
- **Scale In:** −1 instance when CPU < 60% for 5 minutes  
- **Cooldown:** 300 seconds  

---

### 🪣 S3 Lifecycle Policy

- **Day 0–30:** Files remain on EFS (fast access)
- **Day 30–90:** Transition to S3 Standard (backup storage)
- **Day 90+:** Transition to Glacier Deep Archive (cold storage)
