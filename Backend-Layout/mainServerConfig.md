```js
// src/app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import rateLimit from 'express-rate-limit';
import { connectDB } from './config/database';
import { errorHandler } from './middleware/errorHandler';
import authRoutes from './routes/auth';
import universityRoutes from './routes/universities';
import departmentRoutes from './routes/departments';
import courseRoutes from './routes/courses';
import lecturerRoutes from './routes/lecturers';
import studentRoutes from './routes/students';
import contentRoutes from './routes/content';
import quizRoutes from './routes/quizzes';
import certificateRoutes from './routes/certificates';
import chatRoutes from './routes/chat';

const app = express();

// Security middleware
app.use(helmet());
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP, please try again later.'
});
app.use('/api/', limiter);

// Parsing middleware
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Logging
app.use(morgan('combined'));

// Health check
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'OK',
    timestamp: new Date().toISOString(),
    environment: process.env.NODE_ENV || 'development'
  });
});

// API Routes
app.use('/api/auth', authRoutes);
app.use('/api/universities', universityRoutes);
app.use('/api/departments', departmentRoutes);
app.use('/api/courses', courseRoutes);
app.use('/api/lecturers', lecturerRoutes);
app.use('/api/students', studentRoutes);
app.use('/api/content', contentRoutes);
app.use('/api/quizzes', quizRoutes);
app.use('/api/certificates', certificateRoutes);
app.use('/api/chat', chatRoutes);

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({
    success: false,
    message: `Route ${req.originalUrl} not found`
  });
});

// Global error handler
app.use(errorHandler);

const PORT = process.env.PORT || 5000;

const startServer = async () => {
  try {
    await connectDB();
    app.listen(PORT, () => {
      console.log(`🚀 UniSkill+ Backend running on port ${PORT}`);
      console.log(`📊 Environment: ${process.env.NODE_ENV || 'development'}`);
    });
  } catch (error) {
    console.error('Failed to start server:', error);
    process.exit(1);
  }
};

startServer();

export default app;

// src/config/database.ts
import mongoose from 'mongoose';

export const connectDB = async (): Promise<void> => {
  try {
    const mongoURI = process.env.MONGODB_URI || 'mongodb://localhost:27017/uniskill-plus';

    await mongoose.connect(mongoURI);

    console.log('✅ MongoDB Connected Successfully');

    // Handle connection events
    mongoose.connection.on('error', (err) => {
      console.error('MongoDB connection error:', err);
    });

    mongoose.connection.on('disconnected', () => {
      console.log('MongoDB disconnected');
    });

    // Graceful shutdown
    process.on('SIGINT', async () => {
      await mongoose.connection.close();
      console.log('MongoDB connection closed through app termination');
      process.exit(0);
    });

  } catch (error) {
    console.error('❌ MongoDB connection failed:', error);
    process.exit(1);
  }
};

// src/config/environment.ts
import dotenv from 'dotenv';

dotenv.config();

export const config = {
  // Server
  PORT: process.env.PORT || 5000,
  NODE_ENV: process.env.NODE_ENV || 'development',

  // Database
  MONGODB_URI: process.env.MONGODB_URI || 'mongodb://localhost:27017/uniskill-plus',

  // JWT
  JWT_SECRET: process.env.JWT_SECRET || 'your-super-secret-jwt-key',
  JWT_EXPIRE: process.env.JWT_EXPIRE || '7d',

  // Client URL
  CLIENT_URL: process.env.CLIENT_URL || 'http://localhost:3000',

  // File Storage (Supabase)
  SUPABASE_URL: process.env.SUPABASE_URL,
  SUPABASE_ANON_KEY: process.env.SUPABASE_ANON_KEY,
  SUPABASE_SERVICE_KEY: process.env.SUPABASE_SERVICE_KEY,

  // Blockchain (Polygon)
  POLYGON_RPC_URL: process.env.POLYGON_RPC_URL || 'https://rpc-mumbai.maticvigil.com',
  PRIVATE_KEY: process.env.PRIVATE_KEY,

  // AI (Hugging Face)
  HUGGINGFACE_API_KEY: process.env.HUGGINGFACE_API_KEY,

  // Email (Optional - for notifications)
  SMTP_HOST: process.env.SMTP_HOST,
  SMTP_PORT: process.env.SMTP_PORT,
  SMTP_USER: process.env.SMTP_USER,
  SMTP_PASS: process.env.SMTP_PASS,
};

// src/types/index.ts
import { Document } from 'mongoose';

export interface IUser extends Document {
  name: string;
  email: string;
  passwordHash: string;
  role: 'student' | 'lecturer' | 'uni-admin' | 'super-admin';
  universityId?: string;
  departmentId?: string;
  regNumber?: string; // For students
  staffId?: string; // For lecturers
  isVerified: boolean;
  avatar?: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface IUniversity extends Document {
  name: string;
  emailSuffix: string; // e.g., "@unilag.edu.ng"
  description?: string;
  logo?: string;
  approved: boolean;
  location: {
    state: string;
    country: string;
  };
  createdAt: Date;
  updatedAt: Date;
}

export interface IDepartment extends Document {
  name: string;
  code: string; // e.g., "CSC", "ACC"
  universityId: string;
  description?: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface ICourse extends Document {
  name: string;
  code: string; // e.g., "CSC101", "ACC201"
  description: string;
  departmentId: string;
  lecturerId?: string;
  semester: number;
  level: number; // 100, 200, 300, 400
  credits: number;
  isElective: boolean;
  prerequisites: string[]; // Array of course IDs
  learningOutcomes: string[];
  createdAt: Date;
  updatedAt: Date;
}

export interface IContent extends Document {
  courseId: string;
  title: string;
  type: 'video' | 'note' | 'link' | 'assignment';
  content: string; // URL for video/link, text for notes
  description?: string;
  duration?: number; // In minutes
  order: number;
  isPublished: boolean;
  uploadedBy: string; // lecturer ID
  createdAt: Date;
  updatedAt: Date;
}

export interface IQuiz extends Document {
  courseId: string;
  title: string;
  description?: string;
  questions: IQuestion[];
  duration: number; // In minutes
  passingScore: number; // Percentage
  attempts: number; // Max attempts allowed
  isPublished: boolean;
  createdBy: string; // lecturer ID
  createdAt: Date;
  updatedAt: Date;
}

export interface IQuestion {
  question: string;
  options: string[];
  correctAnswer: number; // Index of correct option
  explanation?: string;
  points: number;
}

export interface IQuizSubmission extends Document {
  quizId: string;
  studentId: string;
  answers: number[]; // Array of selected option indices
  score: number;
  passed: boolean;
  timeSpent: number; // In seconds
  attemptNumber: number;
  submittedAt: Date;
}

export interface ICertificate extends Document {
  studentId: string;
  courseId: string;
  quizId?: string;
  type: 'course-completion' | 'quiz-pass' | 'skill-badge';
  title: string;
  description: string;
  blockchainTxHash?: string;
  ipfsHash?: string;
  issuedBy: string; // lecturer/university ID
  validUntil?: Date;
  issuedAt: Date;
}

export interface IChatChannel extends Document {
  name: string;
  type: 'department' | 'course' | 'skill-group';
  universityId: string;
  departmentId?: string;
  courseId?: string;
  description?: string;
  members: string[]; // Array of user IDs
  admins: string[]; // Array of user IDs who can moderate
  isPrivate: boolean;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface IChatMessage extends Document {
  channelId: string;
  senderId: string;
  content: string;
  type: 'text' | 'file' | 'image';
  fileUrl?: string;
  replyTo?: string; // Message ID if replying
  edited: boolean;
  editedAt?: Date;
  sentAt: Date;
}

// Request/Response interfaces
export interface AuthRequest {
  email: string;
  password: string;
}

export interface StudentRegisterRequest extends AuthRequest {
  name: string;
  regNumber: string;
  universityId: string;
  departmentId: string;
  level: number;
}

export interface LecturerRegisterRequest extends AuthRequest {
  name: string;
  staffId: string;
  universityId: string;
  departmentId: string;
  specialization?: string;
}

export interface ApiResponse<T = any> {
  success: boolean;
  message: string;
  data?: T;
  error?: string;
  pagination?: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  };
}

// JWT Payload
export interface JWTPayload {
  userId: string;
  email: string;
  role: string;
  universityId?: string;
  departmentId?: string;
}
```
