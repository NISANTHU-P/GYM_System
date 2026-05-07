# 🗄️ Complete Database Schemas & Relationships

## 📊 Database Overview

Your GYM application uses **MongoDB** with **11 collections** (schemas) that work together to manage the entire gym ecosystem.

---

## 🎯 Collections Summary

| Collection | Purpose | Key Relationships |
|------------|---------|-------------------|
| **User** | Authentication & core user data | Links to all other collections |
| **Trainer** | Trainer-specific data | Linked to User by email |
| **MemberProfile** | Extended member information | References User & Trainer |
| **TrainerProfile** | Extended trainer information | References User & Trainer |
| **WorkoutPlan** | Exercise plans for members | References Trainer & User |
| **DietPlan** | Nutrition plans for members | References Trainer & User |
| **Progress** | Member fitness tracking | References User, Trainer, WorkoutPlan |
| **Attendance** | Member check-in/out records | References User |
| **Payment** | Financial transactions | References User |
| **Complaint** | Support tickets | References User & Trainer |
| **Notification** | System notifications | References User |

---

## 📋 Detailed Schema Definitions

### 1. User Schema (Core Authentication)

**File:** `backend/models/User.js`

```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (required, unique),
  password: String (required, hashed),
  role: String (enum: ['member', 'trainer', 'admin', 'superadmin']),
  isActive: Boolean (default: false),
  assignedTrainer: ObjectId (ref: 'Trainer'),
  membershipStatus: String (enum: ['active', 'inactive', 'suspended', 'expired']),
  membershipStartDate: Date,
  membershipExpiryDate: Date,
  phone: String,
  address: String,
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Has one → MemberProfile (if role = 'member')
- Has one → TrainerProfile (if role = 'trainer')
- Has many → WorkoutPlan (as member)
- Has many → DietPlan (as member)
- Has many → Progress (as member)
- Has many → Attendance (as member)
- Has many → Payment (as member)
- Has many → Complaint (as user)
- Has many → Notification (as user)
- Belongs to → Trainer (via assignedTrainer)

---

### 2. Trainer Schema (Trainer Details)

**File:** `backend/models/Trainer.js`

```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (required, unique),
  password: String (required, hashed),
  phone: String (required),
  specialization: String (enum: [
    'Strength Training', 'Cardio', 'Yoga', 'HIIT', 
    'CrossFit', 'Pilates', 'Boxing', 'General Fitness'
  ]),
  experience: Number (years),
  certification: String (required),
  salary: Number (required),
  status: String (enum: ['active', 'inactive']),
  joinDate: Date,
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Linked to → User (by email)
- Has one → TrainerProfile
- Has many → User (as assignedTrainer)
- Has many → WorkoutPlan (as creator)
- Has many → DietPlan (as creator)
- Has many → Progress (as tracker)
- Has many → Complaint (as subject)

**Note:** Trainer exists in TWO collections:
- **User collection** (for authentication)
- **Trainer collection** (for trainer-specific data)
- Linked by matching email addresses

---

### 3. MemberProfile Schema (Extended Member Info)

**File:** `backend/models/MemberProfile.js`

```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: 'User', required, unique),
  trainerId: ObjectId (ref: 'Trainer'),
  membershipPlan: String (enum: ['Basic', 'Pro', 'Elite']),
  membershipStartDate: Date,
  membershipExpiryDate: Date (required),
  emergencyContact: {
    name: String,
    phone: String,
    relation: String
  },
  fitnessGoals: String,
  medicalHistory: String,
  height: Number (cm),
  initialWeight: Number (kg),
  status: String (enum: ['active', 'inactive', 'suspended', 'expired']),
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Belongs to → User (one-to-one)
- Belongs to → Trainer (many-to-one)

---

### 4. TrainerProfile Schema (Extended Trainer Info)

**File:** `backend/models/TrainerProfile.js`

```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: 'User', required, unique),
  trainerId: ObjectId (ref: 'Trainer', required),
  assignedMembers: [ObjectId] (ref: 'User'),
  specialization: String (required),
  experience: Number (required),
  certification: String (required),
  salary: Number (required),
  joinDate: Date,
  status: String (enum: ['active', 'inactive']),
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Belongs to → User (one-to-one)
- Belongs to → Trainer (one-to-one)
- Has many → User (via assignedMembers array)

---

### 5. WorkoutPlan Schema (Exercise Plans)

**File:** `backend/models/WorkoutPlan.js`

```javascript
{
  _id: ObjectId,
  title: String (required),
  description: String,
  trainerId: ObjectId (ref: 'Trainer', required),
  memberId: ObjectId (ref: 'User', required),
  exercises: [{
    name: String (required),
    sets: Number (required),
    reps: String (required),
    weight: String,
    duration: String,
    restTime: String
  }],
  difficulty: String (enum: ['Beginner', 'Intermediate', 'Advanced']),
  duration: Number (weeks, required),
  status: String (enum: ['active', 'completed', 'paused']),
  startDate: Date,
  endDate: Date,
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Belongs to → Trainer (many-to-one)
- Belongs to → User/Member (many-to-one)
- Has many → Progress (tracking)

---

### 6. DietPlan Schema (Nutrition Plans)

**File:** `backend/models/DietPlan.js`

```javascript
{
  _id: ObjectId,
  title: String (required),
  description: String,
  trainerId: ObjectId (ref: 'Trainer', required),
  memberId: ObjectId (ref: 'User', required),
  meals: [{
    mealType: String (enum: ['Breakfast', 'Lunch', 'Dinner', 'Snack']),
    foodItems: [{
      name: String,
      quantity: String,
      calories: Number,
      protein: Number,
      carbs: Number,
      fats: Number
    }],
    totalCalories: Number,
    notes: String
  }],
  duration: Number (weeks, required),
  dailyCalorieTarget: Number,
  status: String (enum: ['active', 'completed', 'paused']),
  startDate: Date,
  endDate: Date,
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Belongs to → Trainer (many-to-one)
- Belongs to → User/Member (many-to-one)

---

### 7. Progress Schema (Fitness Tracking)

**File:** `backend/models/Progress.js`

```javascript
{
  _id: ObjectId,
  memberId: ObjectId (ref: 'User', required),
  trainerId: ObjectId (ref: 'Trainer'),
  workoutPlanId: ObjectId (ref: 'WorkoutPlan'),
  date: Date,
  weight: Number (kg),
  bodyFat: Number (%),
  muscle: Number (%),
  notes: String,
  measurements: {
    chest: Number (cm),
    waist: Number (cm),
    hips: Number (cm),
    arms: Number (cm),
    thighs: Number (cm)
  },
  workoutCompleted: Boolean,
  exercisesCompleted: [{
    exerciseName: String,
    setsCompleted: Number,
    repsCompleted: String,
    weightUsed: String
  }],
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Belongs to → User/Member (many-to-one)
- Belongs to → Trainer (many-to-one)
- Belongs to → WorkoutPlan (many-to-one)

---

### 8. Attendance Schema (Check-in/out Records)

**File:** `backend/models/Attendance.js`

```javascript
{
  _id: ObjectId,
  memberId: ObjectId (ref: 'User', required),
  date: Date (required),
  checkInTime: Date (required),
  checkOutTime: Date,
  duration: Number (minutes),
  method: String (enum: ['manual', 'qr']),
  status: String (enum: ['present', 'absent', 'late']),
  notes: String,
  createdAt: Date,
  updatedAt: Date
}

// Indexes
Index: { memberId: 1, date: 1 }
```

**Relationships:**
- Belongs to → User/Member (many-to-one)

---

### 9. Payment Schema (Financial Transactions)

**File:** `backend/models/Payment.js`

```javascript
{
  _id: ObjectId,
  memberId: ObjectId (ref: 'User', required),
  amount: Number (required),
  paymentType: String (enum: [
    'membership', 'personal_training', 'supplements', 'other'
  ]),
  membershipPlan: String (enum: ['Basic', 'Pro', 'Elite']),
  paymentMethod: String (enum: ['cash', 'card', 'online', 'upi']),
  transactionId: String,
  status: String (enum: ['pending', 'completed', 'failed', 'refunded']),
  paymentDate: Date,
  dueDate: Date,
  notes: String,
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Belongs to → User/Member (many-to-one)

---

### 10. Complaint Schema (Support Tickets)

**File:** `backend/models/Complaint.js`

```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: 'User', required),
  trainerId: ObjectId (ref: 'Trainer', required),
  subject: String (required),
  message: String (required),
  status: String (enum: ['pending', 'reviewed', 'resolved', 'dismissed']),
  adminResponse: String,
  reviewedBy: ObjectId (ref: 'User'),
  reviewedAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

**Relationships:**
- Belongs to → User (many-to-one)
- Belongs to → Trainer (many-to-one)
- Reviewed by → User/Admin (many-to-one)

---

### 11. Notification Schema (System Alerts)

**File:** `backend/models/Notification.js`

```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: 'User', required),
  type: String (enum: [
    'membership_expiry', 'payment_alert', 'announcement',
    'workout_assigned', 'diet_assigned', 'progress_update',
    'complaint_response'
  ]),
  title: String (required),
  message: String (required),
  isRead: Boolean (default: false),
  relatedId: ObjectId (generic reference),
  priority: String (enum: ['low', 'medium', 'high']),
  createdAt: Date,
  updatedAt: Date
}

// Indexes
Index: { userId: 1, isRead: 1 }
```

**Relationships:**
- Belongs to → User (many-to-one)
- Can reference → Any collection (via relatedId)

---

## 🔗 Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATABASE RELATIONSHIPS                       │
└─────────────────────────────────────────────────────────────────┘

                        ┌──────────────┐
                        │     USER     │
                        │  (Core Auth) │
                        └──────┬───────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
        ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
        │ MemberProfile│ │TrainerProfile│ │ Notification │
        │  (1-to-1)    │ │  (1-to-1)    │ │ (1-to-many)  │
        └──────┬───────┘ └──────┬───────┘ └──────────────┘
               │                │
               │                │
               ▼                ▼
        ┌──────────────┐ ┌──────────────┐
        │   TRAINER    │◄┤ assignedMembers
        │ (Linked by   │ │   (array)    │
        │   email)     │ └──────────────┘
        └──────┬───────┘
               │
               │ Creates & Manages
               │
    ┌──────────┼──────────┬──────────┬──────────┐
    │          │          │          │          │
    ▼          ▼          ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│Workout │ │  Diet  │ │Progress│ │Complaint│ │Attendance│
│  Plan  │ │  Plan  │ │        │ │        │ │         │
└────────┘ └────────┘ └────────┘ └────────┘ └────────┘
    │          │          │          │          │
    └──────────┴──────────┴──────────┴──────────┘
                     │
                     ▼
              ┌──────────────┐
              │   Payment    │
              │              │
              └──────────────┘
```

---

## 🔄 Complete Workflow Diagrams

### Workflow 1: New Member Registration

```
1. USER REGISTERS
   ↓
   POST /api/users/register
   ↓
   Create User document:
   {
     name, email, password (hashed),
     role: 'member',
     isActive: false,  ← Needs admin approval
     membershipStatus: 'inactive'
   }
   ↓
   User created but CANNOT login yet

2. ADMIN APPROVES
   ↓
   PUT /api/admin/approve-user/:userId
   ↓
   Update User:
   {
     isActive: true,
     membershipStatus: 'active',
     membershipStartDate: now,
     membershipExpiryDate: now + 30 days
   }
   ↓
   Create MemberProfile:
   {
     userId: user._id,
     membershipPlan: 'Basic',
     membershipStartDate: now,
     membershipExpiryDate: now + 30 days,
     status: 'active'
   }
   ↓
   Create Notification:
   {
     userId: user._id,
     type: 'announcement',
     title: 'Account Activated',
     message: 'Welcome to the gym!'
   }
   ↓
   Member can now login
```

---

### Workflow 2: Admin Assigns Trainer to Member

```
1. ADMIN SELECTS MEMBER & TRAINER
   ↓
   PUT /api/admin/assign-trainer
   Body: { memberId, trainerId }
   ↓
   
2. FIND DOCUMENTS
   ↓
   Member = User.findById(memberId)
   TrainerUser = User.findById(trainerId)
   Trainer = Trainer.findOne({ email: TrainerUser.email })
   ↓
   
3. UPDATE MEMBER
   ↓
   Member.assignedTrainer = Trainer._id
   Member.save()
   ↓
   
4. UPDATE MEMBER PROFILE
   ↓
   MemberProfile.findOne({ userId: memberId })
   MemberProfile.trainerId = Trainer._id
   MemberProfile.save()
   ↓
   
5. UPDATE TRAINER PROFILE
   ↓
   TrainerProfile.findOne({ trainerId: Trainer._id })
   TrainerProfile.assignedMembers.push(memberId)
   TrainerProfile.save()
   ↓
   
6. CREATE NOTIFICATION
   ↓
   Notification.create({
     userId: memberId,
     type: 'announcement',
     title: 'Trainer Assigned',
     message: `Trainer ${Trainer.name} assigned to you`
   })
   ↓
   Assignment complete!
```

---

### Workflow 3: Trainer Creates Workout Plan

```
1. TRAINER SELECTS MEMBER
   ↓
   POST /api/trainer/workout-plan
   Body: {
     memberId,
     title: "Full Body Strength",
     description: "...",
     difficulty: "Intermediate",
     duration: 8,
     exercises: "Bench Press\nSquats\nDeadlifts"
   }
   ↓
   
2. VERIFY TRAINER
   ↓
   TrainerUser = User.findById(req.user.id)  ← From JWT
   Trainer = Trainer.findOne({ email: TrainerUser.email })
   ↓
   
3. VERIFY MEMBER ASSIGNMENT
   ↓
   Member = User.findOne({
     _id: memberId,
     assignedTrainer: Trainer._id
   })
   If not found → Error: "Member not assigned to you"
   ↓
   
4. PARSE EXERCISES
   ↓
   exercises = "Bench Press\nSquats\nDeadlifts"
   ↓
   exerciseArray = [
     { name: "Bench Press", sets: 3, reps: "10-12", restTime: "60s" },
     { name: "Squats", sets: 3, reps: "10-12", restTime: "60s" },
     { name: "Deadlifts", sets: 3, reps: "10-12", restTime: "60s" }
   ]
   ↓
   
5. CREATE WORKOUT PLAN
   ↓
   WorkoutPlan.create({
     title: "Full Body Strength",
     description: "...",
     trainerId: Trainer._id,  ← Trainer collection ID
     memberId: memberId,       ← User collection ID
     exercises: exerciseArray,
     difficulty: "Intermediate",
     duration: 8,
     status: 'active',
     startDate: now,
     endDate: now + (8 * 7 days)
   })
   ↓
   
6. CREATE NOTIFICATION
   ↓
   Notification.create({
     userId: memberId,
     type: 'workout_assigned',
     title: 'New Workout Plan',
     message: `${Trainer.name} assigned you a workout plan`,
     relatedId: workoutPlan._id
   })
   ↓
   Workout plan created!
```

---

### Workflow 4: Member Views Workout Plans

```
1. MEMBER LOGS IN
   ↓
   JWT token contains: { id: member._id, role: 'member' }
   ↓
   
2. MEMBER NAVIGATES TO WORKOUT PAGE
   ↓
   GET /api/member/workout-plans
   ↓
   
3. FETCH WORKOUT PLANS
   ↓
   WorkoutPlan.find({
     memberId: req.user.id  ← From JWT
   })
   .populate('trainerId', 'name specialization')
   ↓
   
4. RETURN PLANS
   ↓
   [
     {
       _id: "...",
       title: "Full Body Strength",
       description: "...",
       trainerId: {
         _id: "...",
         name: "John Trainer",
         specialization: "Strength Training"
       },
       exercises: [
         { name: "Bench Press", sets: 3, reps: "10-12", ... },
         { name: "Squats", sets: 3, reps: "10-12", ... }
       ],
       difficulty: "Intermediate",
       duration: 8,
       status: "active",
       startDate: "2024-01-01",
       endDate: "2024-02-26"
     }
   ]
   ↓
   
5. DISPLAY ON FRONTEND
   ↓
   Member sees all workout plans with exercises
```

---

### Workflow 5: Trainer Tracks Member Progress

```
1. TRAINER VIEWS MEMBER
   ↓
   GET /api/trainer/members
   ↓
   Returns list of assigned members
   ↓
   
2. TRAINER ADDS PROGRESS ENTRY
   ↓
   POST /api/trainer/progress
   Body: {
     memberId: "...",
     weight: 75,
     bodyFat: 18,
     muscle: 42,
     measurements: {
       chest: 100,
       waist: 85,
       arms: 35
     },
     notes: "Great progress this week!"
   }
   ↓
   
3. CREATE PROGRESS DOCUMENT
   ↓
   TrainerUser = User.findById(req.user.id)
   Trainer = Trainer.findOne({ email: TrainerUser.email })
   ↓
   Progress.create({
     memberId: memberId,
     trainerId: Trainer._id,
     date: now,
     weight: 75,
     bodyFat: 18,
     muscle: 42,
     measurements: { ... },
     notes: "..."
   })
   ↓
   
4. CREATE NOTIFICATION
   ↓
   Notification.create({
     userId: memberId,
     type: 'progress_update',
     title: 'Progress Updated',
     message: 'Your trainer updated your progress',
     relatedId: progress._id
   })
   ↓
   Progress tracked!
```

---

### Workflow 6: Member Checks In (Attendance)

```
1. MEMBER ARRIVES AT GYM
   ↓
   POST /api/attendance
   Body: {
     memberId: member._id,
     method: 'manual'  // or 'qr'
   }
   ↓
   
2. CREATE ATTENDANCE RECORD
   ↓
   Attendance.create({
     memberId: member._id,
     date: today,
     checkInTime: now,
     method: 'manual',
     status: 'present'
   })
   ↓
   
3. MEMBER LEAVES GYM
   ↓
   PUT /api/attendance/checkout/:attendanceId
   ↓
   
4. UPDATE ATTENDANCE
   ↓
   Attendance.findById(attendanceId)
   attendance.checkOutTime = now
   attendance.duration = (checkOutTime - checkInTime) / 60  // minutes
   attendance.save()
   ↓
   Attendance recorded!
```

---

### Workflow 7: Member Makes Payment

```
1. MEMBER INITIATES PAYMENT
   ↓
   POST /api/payments
   Body: {
     memberId: member._id,
     amount: 1500,
     paymentType: 'membership',
     membershipPlan: 'Pro',
     paymentMethod: 'online'
   }
   ↓
   
2. CREATE PAYMENT RECORD
   ↓
   Payment.create({
     memberId: member._id,
     amount: 1500,
     paymentType: 'membership',
     membershipPlan: 'Pro',
     paymentMethod: 'online',
     status: 'pending',
     paymentDate: now,
     transactionId: generateId()
   })
   ↓
   
3. PROCESS PAYMENT
   ↓
   // Payment gateway integration
   ↓
   
4. UPDATE PAYMENT STATUS
   ↓
   Payment.findById(paymentId)
   payment.status = 'completed'
   payment.save()
   ↓
   
5. UPDATE MEMBERSHIP
   ↓
   User.findById(memberId)
   user.membershipExpiryDate = now + 30 days
   user.save()
   ↓
   MemberProfile.findOne({ userId: memberId })
   memberProfile.membershipExpiryDate = now + 30 days
   memberProfile.save()
   ↓
   
6. CREATE NOTIFICATION
   ↓
   Notification.create({
     userId: memberId,
     type: 'payment_alert',
     title: 'Payment Successful',
     message: 'Your membership has been renewed',
     relatedId: payment._id
   })
   ↓
   Payment complete!
```

---

## 🔍 Query Patterns & Examples

### Pattern 1: Get Member with Full Details

```javascript
// Get member with trainer, profile, and workout plans
const member = await User.findById(memberId)
  .populate('assignedTrainer', 'name specialization')
  .lean();

const memberProfile = await MemberProfile.findOne({ userId: memberId });

const workoutPlans = await WorkoutPlan.find({ memberId })
  .populate('trainerId', 'name specialization');

const dietPlans = await DietPlan.find({ memberId })
  .populate('trainerId', 'name');

const progress = await Progress.find({ memberId })
  .sort({ date: -1 })
  .limit(10);

// Complete member data
const fullMemberData = {
  ...member,
  profile: memberProfile,
  workoutPlans,
  dietPlans,
  recentProgress: progress
};
```

---

### Pattern 2: Get Trainer with Assigned Members

```javascript
// Get trainer with all assigned members
const trainerUser = await User.findById(trainerId);
const trainer = await Trainer.findOne({ email: trainerUser.email });

const assignedMembers = await User.find({
  assignedTrainer: trainer._id,
  role: 'member'
}).select('-password');

const workoutPlansCreated = await WorkoutPlan.find({
  trainerId: trainer._id
}).populate('memberId', 'name email');

// Complete trainer data
const fullTrainerData = {
  ...trainer.toObject(),
  assignedMembers,
  workoutPlansCreated
};
```

---

### Pattern 3: Dashboard Statistics

```javascript
// Admin dashboard stats
const stats = {
  totalMembers: await User.countDocuments({ role: 'member', isActive: true }),
  totalTrainers: await User.countDocuments({ role: 'trainer', isActive: true }),
  pendingApprovals: await User.countDocuments({ isActive: false }),
  
  monthlyRevenue: await Payment.aggregate([
    {
      $match: {
        paymentDate: { $gte: startOfMonth },
        status: 'completed'
      }
    },
    {
      $group: {
        _id: null,
        total: { $sum: '$amount' }
      }
    }
  ]),
  
  todayAttendance: await Attendance.countDocuments({
    date: { $gte: startOfDay }
  }),
  
  activeWorkoutPlans: await WorkoutPlan.countDocuments({ status: 'active' })
};
```

---

## 📊 Data Flow Summary

```
USER REGISTRATION
└─> User (inactive)
    └─> Admin Approval
        └─> User (active) + MemberProfile + Notification

TRAINER ASSIGNMENT
└─> Admin selects Member + Trainer
    └─> User.assignedTrainer = Trainer._id
    └─> MemberProfile.trainerId = Trainer._id
    └─> TrainerProfile.assignedMembers.push(memberId)
    └─> Notification created

WORKOUT PLAN CREATION
└─> Trainer creates plan
    └─> WorkoutPlan (trainerId, memberId, exercises)
    └─> Notification created

PROGRESS TRACKING
└─> Trainer adds progress
    └─> Progress (memberId, trainerId, measurements)
    └─> Notification created

ATTENDANCE
└─> Member checks in
    └─> Attendance (memberId, checkInTime)
    └─> Member checks out
        └─> Attendance.checkOutTime + duration

PAYMENT
└─> Member pays
    └─> Payment (memberId, amount, status)
    └─> User.membershipExpiryDate updated
    └─> MemberProfile.membershipExpiryDate updated
    └─> Notification created
```

---

## 🎯 Key Takeaways

1. **User is Central**: All collections reference User as the core entity
2. **Dual Trainer System**: Trainers exist in both User and Trainer collections
3. **Profile Extensions**: MemberProfile and TrainerProfile extend User data
4. **Relationship Tracking**: assignedTrainer links members to trainers
5. **Notification System**: All major actions trigger notifications
6. **Status Management**: Multiple status fields track entity states
7. **Timestamps**: All collections have createdAt and updatedAt
8. **Indexes**: Optimized queries with strategic indexes

---

**Your database is well-structured with clear relationships and efficient workflows!** 🗄️
