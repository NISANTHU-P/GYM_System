# 📋 Database Schema Quick Reference

## 🎯 10 Collections Overview

```
┌─────────────────────────────────────────────────────────────┐
│  1. User          - Core authentication & user data         │
│  2. Trainer       - Trainer-specific information            │
│  3. MemberProfile - Extended member details                 │
│  4. WorkoutPlan   - Exercise plans                          │
│  5. DietPlan      - Nutrition plans                         │
│  6. Progress      - Fitness tracking                        │
│  7. Attendance    - Check-in/out records (auto 1hr)        │
│  8. Payment       - Financial transactions                  │
│  9. Complaint     - Support tickets                         │
│ 10. Notification  - System alerts                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔗 Relationship Map

```
                    USER (Central Hub)
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   MemberProfile     TRAINER        Notification
        │                │                
        │                │                
        │                │ Creates        
        │                │                
    ┌───┴───┬──────┬─────┴───┬──────┐      
    ▼       ▼      ▼         ▼      ▼      
Workout  Diet  Progress Complaint Attendance
  Plan   Plan              (1hr auto)
                    │                      
                    ▼                      
                 Payment                   
```

---

## 📊 Collection Details

### 1. User (Authentication)
```javascript
{
  email: "user@gym.com",
  password: "hashed",
  role: "member|trainer|admin|superadmin",
  isActive: true/false,
  assignedTrainer: ObjectId → Trainer
}
```
**Purpose:** Login, roles, basic info
**Links to:** Everything

---

### 2. Trainer (Trainer Data)
```javascript
{
  email: "trainer@gym.com",
  specialization: "Strength Training",
  experience: 5,
  certification: "ACE Certified",
  salary: 50000
}
```
**Purpose:** Trainer-specific details
**Links to:** User (by email), WorkoutPlan, DietPlan

---

### 3. MemberProfile (Member Details)
```javascript
{
  userId: ObjectId → User,
  trainerId: ObjectId → Trainer,
  membershipPlan: "Basic|Pro|Elite",
  fitnessGoals: "Lose weight",
  height: 175,
  initialWeight: 80
}
```
**Purpose:** Extended member information
**Links to:** User, Trainer

---



### 5. WorkoutPlan (Exercise Plans)
```javascript
{
  trainerId: ObjectId → Trainer,
  memberId: ObjectId → User,
  exercises: [{
    name: "Bench Press",
    sets: 3,
    reps: "10-12"
  }],
  difficulty: "Intermediate",
  duration: 8
}
```
**Purpose:** Workout programs
**Links to:** Trainer, Member

---

### 6. DietPlan (Nutrition Plans)
```javascript
{
  trainerId: ObjectId → Trainer,
  memberId: ObjectId → User,
  meals: [{
    mealType: "Breakfast",
    foodItems: [...]
  }],
  dailyCalorieTarget: 2000
}
```
**Purpose:** Nutrition programs
**Links to:** Trainer, Member

---

### 7. Progress (Fitness Tracking)
```javascript
{
  memberId: ObjectId → User,
  trainerId: ObjectId → Trainer,
  workoutPlanId: ObjectId → WorkoutPlan,
  weight: 75,
  bodyFat: 18,
  measurements: { chest: 100, waist: 85 }
}
```
**Purpose:** Track member progress
**Links to:** Member, Trainer, WorkoutPlan

---

### 7. Attendance (Check-in/out)
```javascript
{
  memberId: ObjectId → User,
  checkInTime: Date,
  checkOutTime: Date, // Auto-set +1hr by trainer
  duration: 60, // Fixed 60 minutes for trainer sessions
  method: "manual|qr",
  status: "present"
}
```
**Purpose:** Track gym visits (auto 1-hour sessions)
**Links to:** Member

---

### 8. Payment (Transactions)
```javascript
{
  memberId: ObjectId → User,
  amount: 1500,
  paymentType: "membership",
  membershipPlan: "Pro",
  status: "completed"
}
```
**Purpose:** Financial records
**Links to:** Member

---

### 9. Complaint (Support)
```javascript
{
  userId: ObjectId → User,
  trainerId: ObjectId → Trainer,
  subject: "Equipment issue",
  status: "pending|resolved",
  adminResponse: "Fixed"
}
```
**Purpose:** Support tickets
**Links to:** User, Trainer, Admin

---

### 10. Notification (Alerts)
```javascript
{
  userId: ObjectId → User,
  type: "workout_assigned",
  title: "New Workout",
  message: "Check your new plan",
  isRead: false
}
```
**Purpose:** System notifications
**Links to:** User

---

## 🔄 Common Workflows

### New Member Flow
```
Register → User (inactive)
         ↓
Admin Approves → User (active) + MemberProfile
         ↓
Admin Assigns Trainer → User.assignedTrainer = Trainer._id
         ↓
Trainer Creates Plan → WorkoutPlan
         ↓
Member Views Plan → GET /api/member/workout-plans
```

### Trainer Workflow
```
Login → JWT with Trainer User ID
      ↓
View Members → Find User where assignedTrainer = Trainer._id
      ↓
Create Workout → WorkoutPlan (trainerId, memberId)
      ↓
Track Progress → Progress (memberId, trainerId)
```

### Payment Flow
```
Member Pays → Payment (pending)
           ↓
Process → Payment (completed)
           ↓
Update Membership → User.membershipExpiryDate
           ↓
Notify → Notification
```

---

## 🎯 Key Relationships

### User ↔ Trainer
```
User (role: 'trainer')
  ↕ (linked by email)
Trainer (trainer data)
```

### Member ↔ Trainer Assignment
```
User (member)
  .assignedTrainer → Trainer._id
```

### Trainer ↔ Plans
```
Trainer._id
  ↓
WorkoutPlan.trainerId
DietPlan.trainerId
Progress.trainerId
```

### Member ↔ Plans
```
User._id (member)
  ↓
WorkoutPlan.memberId
DietPlan.memberId
Progress.memberId
Attendance.memberId
Payment.memberId
```

---

## 📝 Important Notes

1. **Dual Trainer System**
   - User collection (authentication)
   - Trainer collection (data)
   - Linked by email

2. **ID References**
   - User._id for members
   - Trainer._id for trainer operations
   - Always map between them

3. **Attendance System**
   - Trainer marks attendance → Auto 1-hour session
   - checkInTime: current time
   - checkOutTime: current time + 1 hour
   - duration: fixed 60 minutes

4. **Status Fields**
   - User.isActive (account status)
   - User.membershipStatus (membership)
   - WorkoutPlan.status (plan status)
   - Payment.status (transaction)

5. **Timestamps**
   - All collections have createdAt & updatedAt
   - Automatic via Mongoose

6. **Indexes**
   - Attendance: { memberId, date }
   - Notification: { userId, isRead }

---

## 🔍 Quick Queries

### Get Member Full Data
```javascript
User.findById(memberId)
  .populate('assignedTrainer')
MemberProfile.findOne({ userId: memberId })
WorkoutPlan.find({ memberId })
```

### Get Trainer's Members
```javascript
Trainer.findOne({ email: trainerUser.email })
User.find({ assignedTrainer: trainer._id })
```

### Get Dashboard Stats
```javascript
User.countDocuments({ role: 'member', isActive: true })
Payment.aggregate([{ $match: {...}, $group: {...} }])
Attendance.countDocuments({ date: today })
```

---

## ✅ Checklist

- [x] 10 collections defined (removed TrainerProfile)
- [x] Relationships established
- [x] Indexes created
- [x] Timestamps enabled
- [x] Validation rules set
- [x] References configured
- [x] Enums defined
- [x] Workflows documented
- [x] Auto 1-hour attendance system implemented

---

**For detailed information, see DATABASE_SCHEMAS_COMPLETE.md** 📚
