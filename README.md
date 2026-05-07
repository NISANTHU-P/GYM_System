# GYM - Full Stack Fitness Application

A complete full-stack web application built with React (Vite), Node.js, Express, and MongoDB. This application provides user registration, login, and a dashboard for fitness tracking.

## 🚀 Features

- **User Authentication**: Register and login functionality
- **Responsive Design**: Clean, minimal UI that works on all devices
- **Dashboard**: User dashboard with fitness statistics
- **Profile Management**: View and manage user profile
- **Modern Stack**: React with Vite, Express, MongoDB

## 📁 Project Structure

```
GYM/
├── backend/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   └── userController.js
│   ├── models/
│   │   └── User.js
│   ├── routes/
│   │   └── userRoutes.js
│   ├── index.js
│   ├── package.json
│   └── .env
│
└── frontend/
    ├── src/
    │   ├── components/
    │   │   ├── Navbar.jsx
    │   │   ├── Footer.jsx
    │   │   ├── Home.jsx
    │   │   ├── Login.jsx
    │   │   ├── Register.jsx
    │   │   ├── About.jsx
    │   │   ├── Contact.jsx
    │   │   ├── Dashboard.jsx
    │   │   └── Profile.jsx
    │   ├── styles/
    │   │   ├── Navbar.js
    │   │   ├── Footer.js
    │   │   ├── Home.js
    │   │   ├── Login.js
    │   │   ├── Register.js
    │   │   ├── About.js
    │   │   ├── Contact.js
    │   │   ├── Dashboard.js
    │   │   └── Profile.js
    │   ├── services/
    │   │   └── api.js
    │   ├── App.jsx
    │   └── main.jsx
    ├── index.html
    ├── package.json
    ├── vite.config.js
    └── .env
```

## 🛠️ Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** (v18 or higher)
- **npm** or **yarn**
- **MongoDB** (local installation or MongoDB Atlas account)

## 📦 Installation & Setup

### Step 1: Clone or Navigate to Project

```bash
cd GYM
```

### Step 2: Backend Setup

1. Navigate to the backend directory:
```bash
cd backend
```

2. Install dependencies:
```bash
npm install
```

3. Create a `.env` file in the `backend` directory:
```bash
# Copy the example file (if available) or create new
```

4. Add the following to `backend/.env`:
```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/gym-app
NODE_ENV=development
```

**For MongoDB Atlas (Cloud):**
```env
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/gym-app?retryWrites=true&w=majority
```

5. Start the backend server:
```bash
npm start
```

Or for development with auto-reload:
```bash
npm run dev
```

The backend server will run on `http://localhost:5000`

### Step 3: Frontend Setup

1. Open a new terminal and navigate to the frontend directory:
```bash
cd frontend
```

2. Install dependencies:
```bash
npm install
```

3. Create a `.env` file in the `frontend` directory:
```bash
# Copy the example file (if available) or create new
```

4. Add the following to `frontend/.env`:
```env
VITE_API_BASE_URL=http://localhost:5000/api
```

5. Start the frontend development server:
```bash
npm run dev
```

The frontend will run on `http://localhost:3000` (or the port specified in vite.config.js)

## 🎯 Usage

1. **Start MongoDB**: Make sure MongoDB is running on your system
   - Local: MongoDB should be running on `mongodb://localhost:27017`
   - Atlas: Your connection string should be in the `.env` file

2. **Start Backend**: 
   ```bash
   cd backend
   npm start
   ```

3. **Start Frontend** (in a new terminal):
   ```bash
   cd frontend
   npm run dev
   ```

4. **Open Browser**: Navigate to `http://localhost:3000`

## 📡 API Endpoints

### User Registration
- **POST** `/api/users/register`
- **Body**: 
  ```json
  {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password123"
  }
  ```

### User Login
- **POST** `/api/users/login`
- **Body**:
  ```json
  {
    "email": "john@example.com",
    "password": "password123"
  }
  ```

## 🧪 Testing the Application

1. **Register a new user**:
   - Navigate to `/register`
   - Fill in name, email, and password (min 6 characters)
   - Submit the form

2. **Login**:
   - Navigate to `/login`
   - Enter your email and password
   - You'll be redirected to the dashboard upon successful login

3. **View Dashboard**:
   - After login, you'll see your dashboard with statistics

4. **View Profile**:
   - Navigate to `/profile` to see your user information

## 🔧 Technologies Used

### Frontend
- **React 18** - UI library
- **Vite** - Build tool and dev server
- **React Router DOM** - Routing
- **Axios** - HTTP client
- **CSS-in-JS** - Styling approach

### Backend
- **Node.js** - Runtime environment
- **Express** - Web framework
- **MongoDB** - Database
- **Mongoose** - MongoDB ODM
- **bcryptjs** - Password hashing
- **dotenv** - Environment variables
- **CORS** - Cross-origin resource sharing

## 📝 Notes

- Passwords are hashed using bcrypt before storing in the database
- User authentication is handled via localStorage (for demo purposes)
- Images are loaded from Unsplash (free open-source images)
- The application uses functional components only
- All styles are CSS-in-JS (JavaScript objects)

## 🐛 Troubleshooting

### Backend Issues

1. **MongoDB Connection Error**:
   - Ensure MongoDB is running
   - Check your `MONGODB_URI` in `.env`
   - Verify network connectivity for Atlas

2. **Port Already in Use**:
   - Change the `PORT` in `backend/.env`
   - Or kill the process using port 5000

### Frontend Issues

1. **API Connection Error**:
   - Verify backend is running
   - Check `VITE_API_BASE_URL` in `frontend/.env`
   - Ensure CORS is enabled in backend

2. **Build Errors**:
   - Delete `node_modules` and reinstall: `rm -rf node_modules && npm install`

## 📄 License

This project is open source and available for educational purposes.

## 👨‍💻 Development

- All components are functional components
- Styles are separated into `.js` files
- API calls are centralized in `services/api.js`
- Backend follows MVC architecture

---

**Happy Coding! 💪**

