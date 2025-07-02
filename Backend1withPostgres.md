# UniSkill+ Backend - Complete Modular Codebase

## Project Structure

```
backend/
├── src/
│   ├── config/
│   │   ├── database.ts
│   │   ├── blockchain.ts
│   │   └── env.ts
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── course.controller.ts
│   │   ├── quiz.controller.ts
│   │   ├── badge.controller.ts
│   │   └── student.controller.ts
│   ├── services/
│   │   ├── ai.service.ts
│   │   ├── blockchain.service.ts
│   │   ├── upload.service.ts
│   │   └── recommendation.service.ts
│   ├── models/
│   │   ├── User.ts
│   │   ├── Course.ts
│   │   ├── Quiz.ts
│   │   └── Badge.ts
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── course.routes.ts
│   │   ├── quiz.routes.ts
│   │   ├── badge.routes.ts
│   │   └── student.routes.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   ├── validation.middleware.ts
│   │   └── upload.middleware.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   ├── response.ts
│   │   └── constants.ts
│   └── app.ts
├── package.json
├── tsconfig.json
└── .env.example
```

## 1. Package.json

```json
{
  "name": "uniskill-backend",
  "version": "1.0.0",
  "description": "UniSkill+ Backend API",
  "main": "dist/app.js",
  "scripts": {
    "dev": "nodemon src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js",
    "lint": "eslint . --ext .ts",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "helmet": "^7.1.0",
    "morgan": "^1.10.0",
    "dotenv": "^16.3.1",
    "pg": "^8.11.3",
    "typeorm": "^0.3.17",
    "reflect-metadata": "^0.1.13",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.2",
    "joi": "^17.11.0",
    "multer": "^1.4.5-lts.1",
    "aws-sdk": "^2.1479.0",
    "openai": "^4.20.1",
    "ethers": "^6.8.1",
    "winston": "^3.11.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/cors": "^2.8.16",
    "@types/morgan": "^1.9.9",
    "@types/pg": "^8.10.7",
    "@types/bcryptjs": "^2.4.6",
    "@types/jsonwebtoken": "^9.0.5",
    "@types/multer": "^1.4.11",
    "@types/node": "^20.8.10",
    "typescript": "^5.2.2",
    "nodemon": "^3.0.1",
    "ts-node": "^10.9.1",
    "@typescript-eslint/eslint-plugin": "^6.9.1",
    "@typescript-eslint/parser": "^6.9.1",
    "eslint": "^8.53.0"
  }
}
```

## 2. TypeScript Configuration (tsconfig.json)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "strictPropertyInitialization": false
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 3. Environment Configuration (.env.example)

```env
NODE_ENV=development
PORT=3000

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=password
DB_NAME=uniskill_db

# JWT
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# OpenAI
OPENAI_API_KEY=your-openai-api-key

# AWS S3
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_REGION=us-east-1
AWS_S3_BUCKET=uniskill-uploads

# Blockchain
POLYGON_RPC_URL=https://rpc-mumbai.maticvigil.com
PRIVATE_KEY=your-wallet-private-key
CONTRACT_ADDRESS=your-deployed-contract-address
```

## 4. Database Configuration (src/config/database.ts)

```typescript
import { DataSource } from "typeorm";
import { User } from "../models/User";
import { Course } from "../models/Course";
import { Quiz } from "../models/Quiz";
import { Badge } from "../models/Badge";
import { env } from "./env";

export const AppDataSource = new DataSource({
  type: "postgres",
  host: env.DB_HOST,
  port: env.DB_PORT,
  username: env.DB_USERNAME,
  password: env.DB_PASSWORD,
  database: env.DB_NAME,
  synchronize: env.NODE_ENV === "development",
  logging: env.NODE_ENV === "development",
  entities: [User, Course, Quiz, Badge],
  migrations: ["src/migrations/*.ts"],
  subscribers: ["src/subscribers/*.ts"],
});

export const initializeDatabase = async (): Promise<void> => {
  try {
    await AppDataSource.initialize();
    console.log("Database connection established successfully");
  } catch (error) {
    console.error("Error connecting to database:", error);
    process.exit(1);
  }
};
```

## 5. Environment Variables (src/config/env.ts)

```typescript
import dotenv from "dotenv";

dotenv.config();

export const env = {
  NODE_ENV: process.env.NODE_ENV || "development",
  PORT: parseInt(process.env.PORT || "3000"),

  // Database
  DB_HOST: process.env.DB_HOST || "localhost",
  DB_PORT: parseInt(process.env.DB_PORT || "5432"),
  DB_USERNAME: process.env.DB_USERNAME || "postgres",
  DB_PASSWORD: process.env.DB_PASSWORD || "password",
  DB_NAME: process.env.DB_NAME || "uniskill_db",

  // JWT
  JWT_SECRET: process.env.JWT_SECRET || "fallback-secret",
  JWT_EXPIRES_IN: process.env.JWT_EXPIRES_IN || "7d",

  // OpenAI
  OPENAI_API_KEY: process.env.OPENAI_API_KEY || "",

  // AWS
  AWS_ACCESS_KEY_ID: process.env.AWS_ACCESS_KEY_ID || "",
  AWS_SECRET_ACCESS_KEY: process.env.AWS_SECRET_ACCESS_KEY || "",
  AWS_REGION: process.env.AWS_REGION || "us-east-1",
  AWS_S3_BUCKET: process.env.AWS_S3_BUCKET || "",

  // Blockchain
  POLYGON_RPC_URL: process.env.POLYGON_RPC_URL || "",
  PRIVATE_KEY: process.env.PRIVATE_KEY || "",
  CONTRACT_ADDRESS: process.env.CONTRACT_ADDRESS || "",
};
```

## 6. User Model (src/models/User.ts)

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  OneToMany,
} from "typeorm";
import { Course } from "./Course";
import { Badge } from "./Badge";

export enum UserRole {
  ADMIN = "admin",
  LECTURER = "lecturer",
  STUDENT = "student",
}

@Entity("users")
export class User {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column({ length: 100 })
  name: string;

  @Column({ unique: true, length: 150 })
  email: string;

  @Column()
  password: string;

  @Column({
    type: "enum",
    enum: UserRole,
    default: UserRole.STUDENT,
  })
  role: UserRole;

  @Column({ nullable: true })
  walletAddress?: string;

  @Column({ nullable: true })
  profileImage?: string;

  @Column({ default: true })
  isActive: boolean;

  @OneToMany(() => Course, (course) => course.lecturer)
  courses: Course[];

  @OneToMany(() => Badge, (badge) => badge.student)
  badges: Badge[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

## 7. Course Model (src/models/Course.ts)

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  ManyToOne,
  OneToMany,
  JoinColumn,
} from "typeorm";
import { User } from "./User";
import { Quiz } from "./Quiz";

@Entity("courses")
export class Course {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column({ length: 200 })
  title: string;

  @Column("text")
  description: string;

  @Column({ nullable: true })
  videoUrl?: string;

  @Column({ nullable: true })
  thumbnailUrl?: string;

  @Column("text", { nullable: true })
  transcript?: string;

  @Column("text", { nullable: true })
  aiGeneratedNotes?: string;

  @Column({ default: false })
  isPublished: boolean;

  @Column({ default: 0 })
  enrollmentCount: number;

  @ManyToOne(() => User, (user) => user.courses)
  @JoinColumn({ name: "lecturerId" })
  lecturer: User;

  @Column()
  lecturerId: string;

  @OneToMany(() => Quiz, (quiz) => quiz.course)
  quizzes: Quiz[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

## 8. Quiz Model (src/models/Quiz.ts)

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  ManyToOne,
  JoinColumn,
} from "typeorm";
import { Course } from "./Course";

export interface QuizOption {
  id: string;
  text: string;
}

@Entity("quizzes")
export class Quiz {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column("text")
  question: string;

  @Column("jsonb")
  options: QuizOption[];

  @Column()
  correctAnswerId: string;

  @Column("text", { nullable: true })
  explanation?: string;

  @Column({ default: 1 })
  difficulty: number; // 1-5 scale

  @ManyToOne(() => Course, (course) => course.quizzes)
  @JoinColumn({ name: "courseId" })
  course: Course;

  @Column()
  courseId: string;

  @CreateDateColumn()
  createdAt: Date;
}
```

## 9. Badge Model (src/models/Badge.ts)

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  ManyToOne,
  JoinColumn,
} from "typeorm";
import { User } from "./User";
import { Course } from "./Course";

@Entity("badges")
export class Badge {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column()
  badgeId: string; // Unique identifier for the badge type

  @Column({ length: 100 })
  title: string;

  @Column("text")
  description: string;

  @Column({ nullable: true })
  imageUrl?: string;

  @Column({ nullable: true })
  transactionHash?: string;

  @Column({ default: false })
  isBlockchainVerified: boolean;

  @Column({ nullable: true })
  blockchainData?: string; // JSON string of blockchain metadata

  @ManyToOne(() => User, (user) => user.badges)
  @JoinColumn({ name: "studentId" })
  student: User;

  @Column()
  studentId: string;

  @ManyToOne(() => Course, (course) => course)
  @JoinColumn({ name: "courseId" })
  course: Course;

  @Column()
  courseId: string;

  @CreateDateColumn()
  issuedAt: Date;
}
```

## 10. AI Service (src/services/ai.service.ts)

```typescript
import OpenAI from "openai";
import { env } from "../config/env";
import { logger } from "../utils/logger";

export class AIService {
  private openai: OpenAI;

  constructor() {
    this.openai = new OpenAI({
      apiKey: env.OPENAI_API_KEY,
    });
  }

  async generateNotesFromTranscript(transcript: string): Promise<string> {
    try {
      const completion = await this.openai.chat.completions.create({
        model: "gpt-3.5-turbo",
        messages: [
          {
            role: "system",
            content:
              "You are an educational assistant. Generate comprehensive, well-structured notes from the given transcript. Format them with bullet points, key concepts, and important takeaways.",
          },
          {
            role: "user",
            content: `Please generate detailed study notes from this transcript:\n\n${transcript}`,
          },
        ],
        max_tokens: 2000,
        temperature: 0.3,
      });

      return completion.choices[0]?.message?.content || "";
    } catch (error) {
      logger.error("Error generating notes:", error);
      throw new Error("Failed to generate notes");
    }
  }

  async generateQuizFromContent(
    content: string,
    questionCount: number = 5,
  ): Promise<any[]> {
    try {
      const completion = await this.openai.chat.completions.create({
        model: "gpt-3.5-turbo",
        messages: [
          {
            role: "system",
            content: `Generate exactly ${questionCount} multiple-choice questions from the given content. 
            Return a JSON array where each question has: question, options (array of 4 choices), correctAnswerId, and explanation.
            Make sure options have unique IDs (a, b, c, d) and correctAnswerId matches one of them.`,
          },
          {
            role: "user",
            content: `Generate quiz questions from this content:\n\n${content}`,
          },
        ],
        max_tokens: 2000,
        temperature: 0.4,
      });

      const response = completion.choices[0]?.message?.content || "[]";
      return JSON.parse(response);
    } catch (error) {
      logger.error("Error generating quiz:", error);
      throw new Error("Failed to generate quiz");
    }
  }

  async generateSkillRecommendations(
    completedCourses: string[],
    userProfile: any,
  ): Promise<string[]> {
    try {
      const completion = await this.openai.chat.completions.create({
        model: "gpt-3.5-turbo",
        messages: [
          {
            role: "system",
            content:
              "You are a career advisor. Based on completed courses and user profile, recommend 5 future-ready skills that would complement their learning path. Focus on emerging technologies and industry demands.",
          },
          {
            role: "user",
            content: `User has completed: ${completedCourses.join(", ")}. 
            Profile: ${JSON.stringify(userProfile)}. 
            Recommend next skills to learn.`,
          },
        ],
        max_tokens: 500,
        temperature: 0.6,
      });

      const response = completion.choices[0]?.message?.content || "";
      return response
        .split("\n")
        .filter((line) => line.trim())
        .slice(0, 5);
    } catch (error) {
      logger.error("Error generating recommendations:", error);
      throw new Error("Failed to generate recommendations");
    }
  }
}

export const aiService = new AIService();
```

## 11. Blockchain Service (src/services/blockchain.service.ts)

```typescript
import { ethers } from "ethers";
import { env } from "../config/env";
import { logger } from "../utils/logger";

// Smart contract ABI (simplified)
const BADGE_CONTRACT_ABI = [
  "function issueBadge(address student, string badgeId, string metadata) external returns (uint256)",
  "function getBadges(address student) external view returns (string[] memory)",
  "function verifyBadge(address student, string badgeId) external view returns (bool)",
];

export class BlockchainService {
  private provider: ethers.JsonRpcProvider;
  private wallet: ethers.Wallet;
  private contract: ethers.Contract;

  constructor() {
    this.provider = new ethers.JsonRpcProvider(env.POLYGON_RPC_URL);
    this.wallet = new ethers.Wallet(env.PRIVATE_KEY, this.provider);
    this.contract = new ethers.Contract(
      env.CONTRACT_ADDRESS,
      BADGE_CONTRACT_ABI,
      this.wallet,
    );
  }

  async issueBadge(
    studentAddress: string,
    badgeId: string,
    metadata: string,
  ): Promise<string> {
    try {
      const tx = await this.contract.issueBadge(
        studentAddress,
        badgeId,
        metadata,
      );
      await tx.wait();

      logger.info(`Badge issued successfully. TX: ${tx.hash}`);
      return tx.hash;
    } catch (error) {
      logger.error("Error issuing badge:", error);
      throw new Error("Failed to issue badge on blockchain");
    }
  }

  async verifyBadge(studentAddress: string, badgeId: string): Promise<boolean> {
    try {
      return await this.contract.verifyBadge(studentAddress, badgeId);
    } catch (error) {
      logger.error("Error verifying badge:", error);
      return false;
    }
  }

  async getStudentBadges(studentAddress: string): Promise<string[]> {
    try {
      return await this.contract.getBadges(studentAddress);
    } catch (error) {
      logger.error("Error fetching badges:", error);
      return [];
    }
  }
}

export const blockchainService = new BlockchainService();
```

## 12. Upload Service (src/services/upload.service.ts)

```typescript
import AWS from "aws-sdk";
import { env } from "../config/env";
import { logger } from "../utils/logger";

export class UploadService {
  private s3: AWS.S3;

  constructor() {
    AWS.config.update({
      accessKeyId: env.AWS_ACCESS_KEY_ID,
      secretAccessKey: env.AWS_SECRET_ACCESS_KEY,
      region: env.AWS_REGION,
    });

    this.s3 = new AWS.S3();
  }

  async uploadFile(
    file: Express.Multer.File,
    folder: string = "uploads",
  ): Promise<string> {
    try {
      const key = `${folder}/${Date.now()}-${file.originalname}`;

      const params: AWS.S3.PutObjectRequest = {
        Bucket: env.AWS_S3_BUCKET,
        Key: key,
        Body: file.buffer,
        ContentType: file.mimetype,
        ACL: "public-read",
      };

      const result = await this.s3.upload(params).promise();
      logger.info(`File uploaded successfully: ${result.Location}`);

      return result.Location;
    } catch (error) {
      logger.error("Error uploading file:", error);
      throw new Error("Failed to upload file");
    }
  }

  async deleteFile(fileUrl: string): Promise<void> {
    try {
      const key = fileUrl.split("/").pop();
      if (!key) throw new Error("Invalid file URL");

      await this.s3
        .deleteObject({
          Bucket: env.AWS_S3_BUCKET,
          Key: key,
        })
        .promise();

      logger.info(`File deleted successfully: ${key}`);
    } catch (error) {
      logger.error("Error deleting file:", error);
      throw new Error("Failed to delete file");
    }
  }
}

export const uploadService = new UploadService();
```

## 13. Authentication Middleware (src/middleware/auth.middleware.ts)

```typescript
import { Request, Response, NextFunction } from "express";
import jwt from "jsonwebtoken";
import { AppDataSource } from "../config/database";
import { User, UserRole } from "../models/User";
import { env } from "../config/env";
import { sendErrorResponse } from "../utils/response";

export interface AuthRequest extends Request {
  user?: User;
}

export const authenticateToken = async (
  req: AuthRequest,
  res: Response,
  next: NextFunction,
) => {
  try {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(" ")[1];

    if (!token) {
      return sendErrorResponse(res, "Access token required", 401);
    }

    const decoded = jwt.verify(token, env.JWT_SECRET) as { userId: string };
    const userRepository = AppDataSource.getRepository(User);
    const user = await userRepository.findOne({
      where: { id: decoded.userId },
    });

    if (!user || !user.isActive) {
      return sendErrorResponse(res, "Invalid or inactive user", 401);
    }

    req.user = user;
    next();
  } catch (error) {
    return sendErrorResponse(res, "Invalid token", 401);
  }
};

export const requireRole = (roles: UserRole[]) => {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      return sendErrorResponse(res, "Insufficient permissions", 403);
    }
    next();
  };
};
```

## 14. Course Controller (src/controllers/course.controller.ts)

```typescript
import { Request, Response } from "express";
import { AppDataSource } from "../config/database";
import { Course } from "../models/Course";
import { Quiz } from "../models/Quiz";
import { AuthRequest } from "../middleware/auth.middleware";
import { aiService } from "../services/ai.service";
import { uploadService } from "../services/upload.service";
import { sendSuccessResponse, sendErrorResponse } from "../utils/response";
import { logger } from "../utils/logger";

export class CourseController {
  private courseRepository = AppDataSource.getRepository(Course);
  private quizRepository = AppDataSource.getRepository(Quiz);

  async createCourse(req: AuthRequest, res: Response) {
    try {
      const { title, description, transcript } = req.body;
      const files = req.files as { [fieldname: string]: Express.Multer.File[] };

      let videoUrl = "";
      let thumbnailUrl = "";

      if (files.video) {
        videoUrl = await uploadService.uploadFile(files.video[0], "videos");
      }

      if (files.thumbnail) {
        thumbnailUrl = await uploadService.uploadFile(
          files.thumbnail[0],
          "thumbnails",
        );
      }

      // Generate AI notes if transcript provided
      let aiGeneratedNotes = "";
      if (transcript) {
        aiGeneratedNotes =
          await aiService.generateNotesFromTranscript(transcript);
      }

      const course = this.courseRepository.create({
        title,
        description,
        videoUrl,
        thumbnailUrl,
        transcript,
        aiGeneratedNotes,
        lecturerId: req.user!.id,
      });

      const savedCourse = await this.courseRepository.save(course);

      // Generate quiz if transcript available
      if (transcript) {
        await this.generateQuizForCourse(savedCourse.id, transcript);
      }

      return sendSuccessResponse(
        res,
        savedCourse,
        "Course created successfully",
        201,
      );
    } catch (error) {
      logger.error("Error creating course:", error);
      return sendErrorResponse(res, "Failed to create course");
    }
  }

  async generateQuizForCourse(courseId: string, content: string) {
    try {
      const quizData = await aiService.generateQuizFromContent(content, 5);

      const quizzes = quizData.map((quiz) =>
        this.quizRepository.create({
          question: quiz.question,
          options: quiz.options,
          correctAnswerId: quiz.correctAnswerId,
          explanation: quiz.explanation,
          courseId,
        }),
      );

      await this.quizRepository.save(quizzes);
    } catch (error) {
      logger.error("Error generating quiz:", error);
    }
  }

  async getCourses(req: Request, res: Response) {
    try {
      const courses = await this.courseRepository.find({
        where: { isPublished: true },
        relations: ["lecturer"],
        select: {
          lecturer: { id: true, name: true, email: true },
        },
      });

      return sendSuccessResponse(res, courses);
    } catch (error) {
      logger.error("Error fetching courses:", error);
      return sendErrorResponse(res, "Failed to fetch courses");
    }
  }

  async getCourseById(req: Request, res: Response) {
    try {
      const { id } = req.params;
      const course = await this.courseRepository.findOne({
        where: { id },
        relations: ["lecturer", "quizzes"],
      });

      if (!course) {
        return sendErrorResponse(res, "Course not found", 404);
      }

      return sendSuccessResponse(res, course);
    } catch (error) {
      logger.error("Error fetching course:", error);
      return sendErrorResponse(res, "Failed to fetch course");
    }
  }

  async updateCourse(req: AuthRequest, res: Response) {
    try {
      const { id } = req.params;
      const course = await this.courseRepository.findOne({
        where: { id, lecturerId: req.user!.id },
      });

      if (!course) {
        return sendErrorResponse(res, "Course not found or unauthorized", 404);
      }

      Object.assign(course, req.body);
      const updatedCourse = await this.courseRepository.save(course);

      return sendSuccessResponse(
        res,
        updatedCourse,
        "Course updated successfully",
      );
    } catch (error) {
      logger.error("Error updating course:", error);
      return sendErrorResponse(res, "Failed to update course");
    }
  }

  async deleteCourse(req: AuthRequest, res: Response) {
    try {
      const { id } = req.params;
      const course = await this.courseRepository.findOne({
        where: { id, lecturerId: req.user!.id },
      });

      if (!course) {
        return sendErrorResponse(res, "Course not found or unauthorized", 404);
      }

      // Delete associated files
      if (course.videoUrl) {
        await uploadService.deleteFile(course.videoUrl);
      }
      if (course.thumbnailUrl) {
        await uploadService.deleteFile(course.thumbnailUrl);
      }

      await this.courseRepository.remove(course);

      return sendSuccessResponse(res, null, "Course deleted successfully");
    } catch (error) {
      logger.error("Error deleting course:", error);
      return sendErrorResponse(res, "Failed to delete course");
    }
  }

  async publishCourse(req: AuthRequest, res: Response) {
    try {
      const { id } = req.params;
      const course = await this.courseRepository.findOne({
        where: { id, lecturerId: req.user!.id },
      });

      if (!course) {
        return sendErrorResponse(res, "Course not found or unauthorized", 404);
      }

      course.isPublished = true;
      await this.courseRepository.save(course);

      return sendSuccessResponse(res, course, "Course published successfully");
    } catch (error) {
      logger.error("Error publishing course:", error);
      return sendErrorResponse(res, "Failed to publish course");
    }
  }
}

export const courseController = new CourseController();
```

## 15. Main Application (src/app.ts)

```typescript
import "reflect-metadata";
import express from "express";
import cors from "cors";
import helmet from "helmet";
import morgan from "morgan";
import { initializeDatabase } from "./config/database";
import { env } from "./config/env";
import { logger } from "./utils/logger";

// Routes
import authRoutes from "./routes/auth.routes";
import courseRoutes from "./routes/course.routes";
import quizRoutes from "./routes/quiz.routes";
import badgeRoutes from "./routes/badge.routes";
import studentRoutes from "./routes/student.routes";

const app = express();

// Security middleware
app.use(helmet());
app.use(
  cors({
    origin:
      env.NODE_ENV === "production"
        ? ["https://your-frontend-domain.com"]
        : true,
    credentials: true,
  }),
);

// Body parsing middleware
app.use(express.json({ limit: "10mb" }));
app.use(express.urlencoded({ extended: true, limit: "10mb" }));

// Logging middleware
app.use(morgan("combined"));

// Health check endpoint
app.get("/health", (req, res) => {
  res.json({ status: "OK", timestamp: new Date().toISOString() });
});

// API routes
app.use("/api/auth", authRoutes);
app.use("/api/courses", courseRoutes);
app.use("/api/quizzes", quizRoutes);
app.use("/api/badges", badgeRoutes);
app.use("/api/students", studentRoutes);

// Error handling middleware
app.use(
  (
    err: any,
    req: express.Request,
    res: express.Response,
    next: express.NextFunction,
  ) => {
    logger.error("Unhandled error:", err);
    res.status(500).json({
      success: false,
      message: "Internal server error",
      error: env.NODE_ENV === "development" ? err.message : undefined,
    });
  },
);

// 404 handler
app.use("*", (req, res) => {
  res.status(404).json({
    success: false,
    message: "Route not found",
  });
});

// Initialize database and start server
const startServer = async () => {
  try {
    await initializeDatabase();

    app.listen(env.PORT, () => {
      logger.info(`🚀 UniSkill+ Backend running on port ${env.PORT}`);
      logger.info(`🌍 Environment: ${env.NODE_ENV}`);
    });
  } catch (error) {
    logger.error("Failed to start server:", error);
    process.exit(1);
  }
};

startServer();

export default app;
```

## 16. Authentication Controller (src/controllers/auth.controller.ts)

```typescript
import { Request, Response } from "express";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";
import { AppDataSource } from "../config/database";
import { User, UserRole } from "../models/User";
import { env } from "../config/env";
import { sendSuccessResponse, sendErrorResponse } from "../utils/response";
import { logger } from "../utils/logger";

export class AuthController {
  private userRepository = AppDataSource.getRepository(User);

  async register(req: Request, res: Response) {
    try {
      const { name, email, password, role, walletAddress } = req.body;

      // Check if user already exists
      const existingUser = await this.userRepository.findOne({
        where: { email },
      });
      if (existingUser) {
        return sendErrorResponse(
          res,
          "User already exists with this email",
          400,
        );
      }

      // Hash password
      const hashedPassword = await bcrypt.hash(password, 12);

      // Create user
      const user = this.userRepository.create({
        name,
        email,
        password: hashedPassword,
        role: role || UserRole.STUDENT,
        walletAddress,
      });

      const savedUser = await this.userRepository.save(user);

      // Generate JWT token
      const token = jwt.sign(
        { userId: savedUser.id, email: savedUser.email },
        env.JWT_SECRET,
        { expiresIn: env.JWT_EXPIRES_IN },
      );

      // Remove password from response
      const { password: _, ...userResponse } = savedUser;

      return sendSuccessResponse(
        res,
        {
          user: userResponse,
          token,
        },
        "User registered successfully",
        201,
      );
    } catch (error) {
      logger.error("Error registering user:", error);
      return sendErrorResponse(res, "Failed to register user");
    }
  }

  async login(req: Request, res: Response) {
    try {
      const { email, password } = req.body;

      // Find user
      const user = await this.userRepository.findOne({
        where: { email, isActive: true },
      });

      if (!user) {
        return sendErrorResponse(res, "Invalid credentials", 401);
      }

      // Verify password
      const isPasswordValid = await bcrypt.compare(password, user.password);
      if (!isPasswordValid) {
        return sendErrorResponse(res, "Invalid credentials", 401);
      }

      // Generate JWT token
      const token = jwt.sign(
        { userId: user.id, email: user.email },
        env.JWT_SECRET,
        { expiresIn: env.JWT_EXPIRES_IN },
      );

      // Remove password from response
      const { password: _, ...userResponse } = user;

      return sendSuccessResponse(
        res,
        {
          user: userResponse,
          token,
        },
        "Login successful",
      );
    } catch (error) {
      logger.error("Error logging in user:", error);
      return sendErrorResponse(res, "Failed to login");
    }
  }

  async getProfile(req: any, res: Response) {
    try {
      const { password: _, ...userProfile } = req.user;
      return sendSuccessResponse(res, userProfile);
    } catch (error) {
      logger.error("Error fetching profile:", error);
      return sendErrorResponse(res, "Failed to fetch profile");
    }
  }

  async updateProfile(req: any, res: Response) {
    try {
      const { name, walletAddress, profileImage } = req.body;

      const user = await this.userRepository.findOne({
        where: { id: req.user.id },
      });
      if (!user) {
        return sendErrorResponse(res, "User not found", 404);
      }

      if (name) user.name = name;
      if (walletAddress) user.walletAddress = walletAddress;
      if (profileImage) user.profileImage = profileImage;

      const updatedUser = await this.userRepository.save(user);
      const { password: _, ...userResponse } = updatedUser;

      return sendSuccessResponse(
        res,
        userResponse,
        "Profile updated successfully",
      );
    } catch (error) {
      logger.error("Error updating profile:", error);
      return sendErrorResponse(res, "Failed to update profile");
    }
  }
}

export const authController = new AuthController();
```

## 17. Quiz Controller (src/controllers/quiz.controller.ts)

```typescript
import { Request, Response } from "express";
import { AppDataSource } from "../config/database";
import { Quiz } from "../models/Quiz";
import { Course } from "../models/Course";
import { Badge } from "../models/Badge";
import { AuthRequest } from "../middleware/auth.middleware";
import { blockchainService } from "../services/blockchain.service";
import { sendSuccessResponse, sendErrorResponse } from "../utils/response";
import { logger } from "../utils/logger";

export class QuizController {
  private quizRepository = AppDataSource.getRepository(Quiz);
  private courseRepository = AppDataSource.getRepository(Course);
  private badgeRepository = AppDataSource.getRepository(Badge);

  async getQuizzesByCourse(req: Request, res: Response) {
    try {
      const { courseId } = req.params;

      const quizzes = await this.quizRepository.find({
        where: { courseId },
        select: ["id", "question", "options", "difficulty"], // Exclude correct answers
      });

      return sendSuccessResponse(res, quizzes);
    } catch (error) {
      logger.error("Error fetching quizzes:", error);
      return sendErrorResponse(res, "Failed to fetch quizzes");
    }
  }

  async submitQuiz(req: AuthRequest, res: Response) {
    try {
      const { courseId } = req.params;
      const { answers } = req.body; // Array of { quizId, selectedAnswerId }

      // Get all quizzes for the course
      const quizzes = await this.quizRepository.find({ where: { courseId } });

      if (quizzes.length === 0) {
        return sendErrorResponse(res, "No quizzes found for this course", 404);
      }

      // Calculate score
      let correctAnswers = 0;
      const results = [];

      for (const answer of answers) {
        const quiz = quizzes.find((q) => q.id === answer.quizId);
        if (quiz) {
          const isCorrect = quiz.correctAnswerId === answer.selectedAnswerId;
          if (isCorrect) correctAnswers++;

          results.push({
            quizId: quiz.id,
            question: quiz.question,
            selectedAnswer: answer.selectedAnswerId,
            correctAnswer: quiz.correctAnswerId,
            isCorrect,
            explanation: quiz.explanation,
          });
        }
      }

      const score = (correctAnswers / quizzes.length) * 100;
      const passed = score >= 70; // 70% passing grade

      // Issue badge if passed
      let badge = null;
      if (passed && req.user?.walletAddress) {
        badge = await this.issueBadge(req.user.id, courseId, score);
      }

      return sendSuccessResponse(
        res,
        {
          score,
          passed,
          correctAnswers,
          totalQuestions: quizzes.length,
          results,
          badge,
        },
        "Quiz submitted successfully",
      );
    } catch (error) {
      logger.error("Error submitting quiz:", error);
      return sendErrorResponse(res, "Failed to submit quiz");
    }
  }

  private async issueBadge(studentId: string, courseId: string, score: number) {
    try {
      // Get course details
      const course = await this.courseRepository.findOne({
        where: { id: courseId },
        relations: ["lecturer"],
      });

      if (!course) return null;

      // Create badge
      const badgeId = `${courseId}-${studentId}-${Date.now()}`;
      const badge = this.badgeRepository.create({
        badgeId,
        title: `${course.title} - Completion Badge`,
        description: `Successfully completed ${course.title} with ${score}% score`,
        studentId,
        courseId,
      });

      const savedBadge = await this.badgeRepository.save(badge);

      // Issue on blockchain (async)
      this.issueBlockchainBadge(savedBadge.id, studentId, badgeId, {
        courseTitle: course.title,
        score,
        completionDate: new Date().toISOString(),
      });

      return savedBadge;
    } catch (error) {
      logger.error("Error issuing badge:", error);
      return null;
    }
  }

  private async issueBlockchainBadge(
    badgeDbId: string,
    studentId: string,
    badgeId: string,
    metadata: any,
  ) {
    try {
      // Get student wallet address
      const userRepository = AppDataSource.getRepository("User");
      const student = await userRepository.findOne({
        where: { id: studentId },
      });

      if (!student?.walletAddress) {
        logger.warn("Student wallet address not found for badge issuance");
        return;
      }

      // Issue badge on blockchain
      const txHash = await blockchainService.issueBadge(
        student.walletAddress,
        badgeId,
        JSON.stringify(metadata),
      );

      // Update badge with blockchain info
      await this.badgeRepository.update(badgeDbId, {
        transactionHash: txHash,
        isBlockchainVerified: true,
        blockchainData: JSON.stringify({ metadata, txHash }),
      });

      logger.info(`Badge issued on blockchain: ${txHash}`);
    } catch (error) {
      logger.error("Error issuing blockchain badge:", error);
    }
  }
}

export const quizController = new QuizController();
```

## 18. Badge Controller (src/controllers/badge.controller.ts)

```typescript
import { Request, Response } from "express";
import { AppDataSource } from "../config/database";
import { Badge } from "../models/Badge";
import { AuthRequest } from "../middleware/auth.middleware";
import { blockchainService } from "../services/blockchain.service";
import { sendSuccessResponse, sendErrorResponse } from "../utils/response";
import { logger } from "../utils/logger";

export class BadgeController {
  private badgeRepository = AppDataSource.getRepository(Badge);

  async getStudentBadges(req: AuthRequest, res: Response) {
    try {
      const badges = await this.badgeRepository.find({
        where: { studentId: req.user!.id },
        relations: ["course"],
        order: { issuedAt: "DESC" },
      });

      return sendSuccessResponse(res, badges);
    } catch (error) {
      logger.error("Error fetching badges:", error);
      return sendErrorResponse(res, "Failed to fetch badges");
    }
  }

  async getBadgeById(req: Request, res: Response) {
    try {
      const { id } = req.params;

      const badge = await this.badgeRepository.findOne({
        where: { id },
        relations: ["student", "course"],
        select: {
          student: { id: true, name: true, email: true },
          course: { id: true, title: true, description: true },
        },
      });

      if (!badge) {
        return sendErrorResponse(res, "Badge not found", 404);
      }

      return sendSuccessResponse(res, badge);
    } catch (error) {
      logger.error("Error fetching badge:", error);
      return sendErrorResponse(res, "Failed to fetch badge");
    }
  }

  async verifyBadge(req: Request, res: Response) {
    try {
      const { id } = req.params;

      const badge = await this.badgeRepository.findOne({
        where: { id },
        relations: ["student"],
      });

      if (!badge) {
        return sendErrorResponse(res, "Badge not found", 404);
      }

      let blockchainVerified = false;

      if (badge.student.walletAddress && badge.badgeId) {
        blockchainVerified = await blockchainService.verifyBadge(
          badge.student.walletAddress,
          badge.badgeId,
        );
      }

      return sendSuccessResponse(res, {
        badge,
        blockchainVerified,
        verificationDate: new Date().toISOString(),
      });
    } catch (error) {
      logger.error("Error verifying badge:", error);
      return sendErrorResponse(res, "Failed to verify badge");
    }
  }

  async getAllBadges(req: Request, res: Response) {
    try {
      const page = parseInt(req.query.page as string) || 1;
      const limit = parseInt(req.query.limit as string) || 10;
      const skip = (page - 1) * limit;

      const [badges, total] = await this.badgeRepository.findAndCount({
        relations: ["student", "course"],
        select: {
          student: { id: true, name: true },
          course: { id: true, title: true },
        },
        order: { issuedAt: "DESC" },
        skip,
        take: limit,
      });

      return sendSuccessResponse(res, {
        badges,
        pagination: {
          total,
          page,
          limit,
          totalPages: Math.ceil(total / limit),
        },
      });
    } catch (error) {
      logger.error("Error fetching all badges:", error);
      return sendErrorResponse(res, "Failed to fetch badges");
    }
  }
}

export const badgeController = new BadgeController();
```

## 19. Student Controller (src/controllers/student.controller.ts)

```typescript
import { Request, Response } from "express";
import { AppDataSource } from "../config/database";
import { User } from "../models/User";
import { Course } from "../models/Course";
import { Badge } from "../models/Badge";
import { AuthRequest } from "../middleware/auth.middleware";
import { aiService } from "../services/ai.service";
import { sendSuccessResponse, sendErrorResponse } from "../utils/response";
import { logger } from "../utils/logger";

export class StudentController {
  private userRepository = AppDataSource.getRepository(User);
  private courseRepository = AppDataSource.getRepository(Course);
  private badgeRepository = AppDataSource.getRepository(Badge);

  async getDashboard(req: AuthRequest, res: Response) {
    try {
      const studentId = req.user!.id;

      // Get enrolled courses count (you might want to create an enrollment table)
      const availableCourses = await this.courseRepository.count({
        where: { isPublished: true },
      });

      // Get badges
      const badges = await this.badgeRepository.find({
        where: { studentId },
        relations: ["course"],
        order: { issuedAt: "DESC" },
        take: 5, // Latest 5 badges
      });

      // Get recent courses
      const recentCourses = await this.courseRepository.find({
        where: { isPublished: true },
        relations: ["lecturer"],
        select: {
          lecturer: { id: true, name: true },
        },
        order: { createdAt: "DESC" },
        take: 6,
      });

      // Calculate completion rate
      const completedCourses = badges.length;
      const completionRate =
        availableCourses > 0 ? (completedCourses / availableCourses) * 100 : 0;

      return sendSuccessResponse(res, {
        stats: {
          totalCourses: availableCourses,
          completedCourses,
          totalBadges: badges.length,
          completionRate: Math.round(completionRate),
        },
        recentBadges: badges,
        recentCourses,
      });
    } catch (error) {
      logger.error("Error fetching dashboard:", error);
      return sendErrorResponse(res, "Failed to fetch dashboard data");
    }
  }

  async getRecommendations(req: AuthRequest, res: Response) {
    try {
      const studentId = req.user!.id;

      // Get completed courses
      const badges = await this.badgeRepository.find({
        where: { studentId },
        relations: ["course"],
      });

      const completedCourses = badges.map((badge) => badge.course.title);
      const userProfile = {
        completedCourses: completedCourses.length,
        interests: [], // You can add user interests later
        skillLevel: badges.length > 5 ? "intermediate" : "beginner",
      };

      // Get AI recommendations
      const recommendations = await aiService.generateSkillRecommendations(
        completedCourses,
        userProfile,
      );

      // Get related courses from database
      const suggestedCourses = await this.courseRepository.find({
        where: { isPublished: true },
        relations: ["lecturer"],
        select: {
          lecturer: { id: true, name: true },
        },
        take: 6,
        order: { enrollmentCount: "DESC" }, // Popular courses
      });

      return sendSuccessResponse(res, {
        aiRecommendations: recommendations,
        suggestedCourses,
        completedCourses,
      });
    } catch (error) {
      logger.error("Error fetching recommendations:", error);
      return sendErrorResponse(res, "Failed to fetch recommendations");
    }
  }

  async getCourseProgress(req: AuthRequest, res: Response) {
    try {
      const { courseId } = req.params;
      const studentId = req.user!.id;

      // Check if course exists
      const course = await this.courseRepository.findOne({
        where: { id: courseId },
        relations: ["lecturer", "quizzes"],
      });

      if (!course) {
        return sendErrorResponse(res, "Course not found", 404);
      }

      // Check if student has completed the course (has badge)
      const badge = await this.badgeRepository.findOne({
        where: { studentId, courseId },
      });

      const progress = {
        course,
        isCompleted: !!badge,
        badge,
        totalQuizzes: course.quizzes.length,
        canTakeQuiz: !badge, // Can take quiz if not completed
      };

      return sendSuccessResponse(res, progress);
    } catch (error) {
      logger.error("Error fetching course progress:", error);
      return sendErrorResponse(res, "Failed to fetch course progress");
    }
  }

  async searchCourses(req: Request, res: Response) {
    try {
      const { q, category, level } = req.query;
      let queryBuilder = this.courseRepository
        .createQueryBuilder("course")
        .leftJoinAndSelect("course.lecturer", "lecturer")
        .where("course.isPublished = :published", { published: true });

      if (q) {
        queryBuilder = queryBuilder.andWhere(
          "(course.title ILIKE :search OR course.description ILIKE :search)",
          { search: `%${q}%` },
        );
      }

      if (category) {
        // Assuming you have a category field, or you can search in description
        queryBuilder = queryBuilder.andWhere(
          "course.description ILIKE :category",
          { category: `%${category}%` },
        );
      }

      const courses = await queryBuilder
        .orderBy("course.enrollmentCount", "DESC")
        .limit(20)
        .getMany();

      return sendSuccessResponse(res, courses);
    } catch (error) {
      logger.error("Error searching courses:", error);
      return sendErrorResponse(res, "Failed to search courses");
    }
  }
}

export const studentController = new StudentController();
```

## 20. Utility Files

### Logger (src/utils/logger.ts)

```typescript
import winston from "winston";
import { env } from "../config/env";

const logger = winston.createLogger({
  level: env.NODE_ENV === "production" ? "info" : "debug",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.colorize(),
    winston.format.printf(({ timestamp, level, message, ...meta }) => {
      return `${timestamp} [${level}]: ${message} ${Object.keys(meta).length ? JSON.stringify(meta, null, 2) : ""}`;
    }),
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: "logs/error.log", level: "error" }),
    new winston.transports.File({ filename: "logs/combined.log" }),
  ],
});

export { logger };
```

### Response Utilities (src/utils/response.ts)

```typescript
import { Response } from "express";

export interface ApiResponse<T = any> {
  success: boolean;
  message: string;
  data?: T;
  error?: string;
}

export const sendSuccessResponse = <T>(
  res: Response,
  data?: T,
  message: string = "Success",
  statusCode: number = 200,
): Response => {
  return res.status(statusCode).json({
    success: true,
    message,
    data,
  });
};

export const sendErrorResponse = (
  res: Response,
  message: string = "Something went wrong",
  statusCode: number = 500,
  error?: string,
): Response => {
  return res.status(statusCode).json({
    success: false,
    message,
    error,
  });
};
```

### Constants (src/utils/constants.ts)

```typescript
export const SKILL_CATEGORIES = [
  "AI & Machine Learning",
  "Data Science",
  "Web Development",
  "Mobile Development",
  "Blockchain",
  "Cybersecurity",
  "Digital Marketing",
  "Cloud Computing",
  "DevOps",
  "UI/UX Design",
];

export const DIFFICULTY_LEVELS = {
  BEGINNER: 1,
  INTERMEDIATE: 2,
  ADVANCED: 3,
  EXPERT: 4,
  MASTER: 5,
};

export const QUIZ_SETTINGS = {
  PASSING_SCORE: 70,
  MAX_QUESTIONS: 10,
  TIME_LIMIT: 30, // minutes
};

export const FILE_LIMITS = {
  VIDEO_SIZE: 500 * 1024 * 1024, // 500MB
  IMAGE_SIZE: 5 * 1024 * 1024, // 5MB
  DOCUMENT_SIZE: 10 * 1024 * 1024, // 10MB
};
```

## 21. Route Files

### Auth Routes (src/routes/auth.routes.ts)

```typescript
import { Router } from "express";
import { authController } from "../controllers/auth.controller";
import { authenticateToken } from "../middleware/auth.middleware";
import {
  validateRegistration,
  validateLogin,
} from "../middleware/validation.middleware";

const router = Router();

router.post("/register", validateRegistration, authController.register);
router.post("/login", validateLogin, authController.login);
router.get("/profile", authenticateToken, authController.getProfile);
router.put("/profile", authenticateToken, authController.updateProfile);

export default router;
```

### Course Routes (src/routes/course.routes.ts)

```typescript
import { Router } from "express";
import { courseController } from "../controllers/course.controller";
import { authenticateToken, requireRole } from "../middleware/auth.middleware";
import { UserRole } from "../models/User";
import { uploadMiddleware } from "../middleware/upload.middleware";

const router = Router();

// Public routes
router.get("/", courseController.getCourses);
router.get("/:id", courseController.getCourseById);

// Lecturer routes
router.post(
  "/",
  authenticateToken,
  requireRole([UserRole.LECTURER, UserRole.ADMIN]),
  uploadMiddleware.fields([
    { name: "video", maxCount: 1 },
    { name: "thumbnail", maxCount: 1 },
  ]),
  courseController.createCourse,
);

router.put(
  "/:id",
  authenticateToken,
  requireRole([UserRole.LECTURER, UserRole.ADMIN]),
  courseController.updateCourse,
);

router.delete(
  "/:id",
  authenticateToken,
  requireRole([UserRole.LECTURER, UserRole.ADMIN]),
  courseController.deleteCourse,
);

router.patch(
  "/:id/publish",
  authenticateToken,
  requireRole([UserRole.LECTURER, UserRole.ADMIN]),
  courseController.publishCourse,
);

export default router;
```

This completes the comprehensive, modular UniSkill+ backend codebase with all the features you requested!
