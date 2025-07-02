```js
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import User from '../models/User';
import { JWTPayload } from '../types';
import { config } from '../config/environment';

export interface AuthRequest extends Request {
  user?: any;
}

export const authenticate = async (req: AuthRequest, res: Response, next: NextFunction) => {
  try {
    const token = req.header('Authorization')?.replace('Bearer ', '');

    if (!token) {
      return res.status(401).json({
        success: false,
        message: 'Access denied. No token provided.'
      });
    }

    const decoded = jwt.verify(token, config.JWT_SECRET) as JWTPayload;
    const user = await User.findById(decoded.userId).select('-passwordHash');

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid token. User not found.'
      });
    }

    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({
      success: false,
      message: 'Invalid token.'
    });
  }
};

export const authorize = (...roles: string[]) => {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({
        success: false,
        message: 'Access denied. Authentication required.'
      });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        success: false,
        message: 'Access denied. Insufficient permissions.'
      });
    }

    next();
  };
};

// Check if user belongs to the same university
export const checkUniversityAccess = (req: AuthRequest, res: Response, next: NextFunction) => {
  const { universityId } = req.params;

  if (!req.user) {
    return res.status(401).json({
      success: false,
      message: 'Authentication required.'
    });
  }

  // Super admin can access any university
  if (req.user.role === 'super-admin') {
    return next();
  }

  // Check if user belongs to the university
  if (req.user.universityId?.toString() !== universityId) {
    return res.status(403).json({
      success: false,
      message: 'Access denied. You can only access your own university data.'
    });
  }

  next();
};

// Check if lecturer owns the course or is admin
export const checkCourseAccess = async (req: AuthRequest, res: Response, next: NextFunction) => {
  try {
    const { courseId } = req.params;

    if (!req.user) {
      return res.status(401).json({
        success: false,
        message: 'Authentication required.'
      });
    }

    // Super admin and uni-admin can access any course
    if (['super-admin', 'uni-admin'].includes(req.user.role)) {
      return next();
    }

    // For lecturers, check if they own the course
    if (req.user.role === 'lecturer') {
      const Course = require('../models/Course').default;
      const course = await Course.findById(courseId);

      if (!course) {
        return res.status(404).json({
          success: false,
          message: 'Course not found.'
        });
      }

      if (course.lecturerId?.toString() !== req.user._id.toString()) {
        return res.status(403).json({
          success: false,
          message: 'Access denied. You can only manage your own courses.'
        });
      }
    }

    next();
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Server error while checking course access.'
    });
  }
};

// src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { Error as MongooseError } from 'mongoose';

export interface CustomError extends Error {
  statusCode?: number;
  code?: number;
  keyValue?: any;
  errors?: any;
}

export const errorHandler = (
  err: CustomError,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  let error = { ...err };
  error.message = err.message;

  console.error('Error:', err);

  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    const message = 'Resource not found';
    error = { ...error, message, statusCode: 404 };
  }

  // Mongoose duplicate key
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue || {})[0];
    const message = `${field} already exists`;
    error = { ...error, message, statusCode: 400 };
  }

  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const mongooseErr = err as MongooseError.ValidationError;
    const message = Object.values(mongooseErr.errors).map(val => val.message).join(', ');
    error = { ...error, message, statusCode: 400 };
  }

  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    const message = 'Invalid token';
    error = { ...error, message, statusCode: 401 };
  }

  if (err.name === 'TokenExpiredError') {
    const message = 'Token expired';
    error = { ...error, message, statusCode: 401 };
  }

  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || 'Server Error',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};

// src/middleware/validation.ts
import { Request, Response, NextFunction } from 'express';
import { body, param, query, validationResult } from 'express-validator';

export const handleValidationErrors = (req: Request, res: Response, next: NextFunction) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      errors: errors.array()
    });
  }

  next();
};

// Auth validation rules
export const registerStudentValidation = [
  body('name').trim().isLength({ min: 2, max: 100 }).withMessage('Name must be between 2 and 100 characters'),
  body('email').isEmail().normalizeEmail().withMessage('Please enter a valid email'),
  body('password').isLength({ min: 6 }).withMessage('Password must be at least 6 characters'),
  body('regNumber').trim().isLength({ min: 1 }).withMessage('Registration number is required'),
  body('universityId').isMongoId().withMessage('Valid university ID is required'),
  body('departmentId').isMongoId().withMessage('Valid department ID is required'),
  body('level').isIn([100, 200, 300, 400, 500]).withMessage('Level must be 100, 200, 300, 400, or 500'),
  handleValidationErrors
];

export const registerLecturerValidation = [
  body('name').trim().isLength({ min: 2, max: 100 }).withMessage('Name must be between 2 and 100 characters'),
  body('email').isEmail().normalizeEmail().withMessage('Please enter a valid email'),
  body('password').isLength({ min: 6 }).withMessage('Password must be at least 6 characters'),
  body('staffId').trim().isLength({ min: 1 }).withMessage('Staff ID is required'),
  body('universityId').isMongoId().withMessage('Valid university ID is required'),
  body('departmentId').isMongoId().withMessage('Valid department ID is required'),
  handleValidationErrors
];

export const loginValidation = [
  body('email').isEmail().normalizeEmail().withMessage('Please enter a valid email'),
  body('password').isLength({ min: 1 }).withMessage('Password is required'),
  handleValidationErrors
];

// University validation
export const createUniversityValidation = [
  body('name').trim().isLength({ min: 2, max: 200 }).withMessage('University name must be between 2 and 200 characters'),
  body('emailSuffix').matches(/@[\w.-]+\.\w{2,}$/).withMessage('Please enter a valid email suffix (e.g., @unilag.edu.ng)'),
  body('location.state').trim().isLength({ min: 1 }).withMessage('State is required'),
  body('location.country').optional().trim().isLength({ min: 1 }),
  body('description').optional().isLength({ max: 1000 }).withMessage('Description cannot exceed 1000 characters'),
  handleValidationErrors
];

// Department validation
export const createDepartmentValidation = [
  body('name').trim().isLength({ min: 2, max: 100 }).withMessage('Department name must be between 2 and 100 characters'),
  body('code').trim().isLength({ min: 2, max: 10 }).withMessage('Department code must be between 2 and 10 characters'),
  body('description').optional().isLength({ max: 500 }).withMessage('Description cannot exceed 500 characters'),
  handleValidationErrors
];

// Course validation
export const createCourseValidation = [
  body('name').trim().isLength({ min: 2, max: 200 }).withMessage('Course name must be between 2 and 200 characters'),
  body('code').trim().isLength({ min: 3, max: 15 }).withMessage('Course code must be between 3 and 15 characters'),
  body('description').trim().isLength({ min: 10, max: 2000 }).withMessage('Description must be between 10 and 2000 characters'),
  body('semester').isIn([1, 2]).withMessage('Semester must be 1 or 2'),
  body('level').isIn([100, 200, 300, 400, 500]).withMessage('Level must be 100, 200, 300, 400, or 500'),
  body('credits').isInt({ min: 1, max: 6 }).withMessage('Credits must be between 1 and 6'),
  body('isElective').optional().isBoolean().withMessage('isElective must be a boolean'),
  body('learningOutcomes').optional().isArray().withMessage('Learning outcomes must be an array'),
  handleValidationErrors
];

// Content validation
export const createContentValidation = [
  body('title').trim().isLength({ min: 2, max: 200 }).withMessage('Title must be between 2 and 200 characters'),
  body('type').isIn(['video', 'note', 'link', 'assignment']).withMessage('Type must be video, note, link, or assignment'),
  body('content').trim().isLength({ min: 1 }).withMessage('Content is required'),
  body('description').optional().isLength({ max: 1000 }).withMessage('Description cannot exceed 1000 characters'),
  body('duration').optional().isInt({ min: 0 }).withMessage('Duration must be a positive number'),
  body('order').isInt({ min: 1 }).withMessage('Order must be a positive number'),
  handleValidationErrors
];

// Quiz validation
export const createQuizValidation = [
  body('title').trim().isLength({ min: 2, max: 200 }).withMessage('Title must be between 2 and 200 characters'),
  body('description').optional().isLength({ max: 1000 }).withMessage('Description cannot exceed 1000 characters'),
  body('duration').isInt({ min: 5, max: 180 }).withMessage('Duration must be between 5 and 180 minutes'),
  body('passingScore').isInt({ min: 0, max: 100 }).withMessage('Passing score must be between 0 and 100'),
  body('attempts').isInt({ min: 1, max: 5 }).withMessage('Attempts must be between 1 and 5'),
  body('questions').isArray({ min: 1 }).withMessage('At least one question is required'),
  body('questions.*.question').trim().isLength({ min: 1, max: 1000 }).withMessage('Question text is required and cannot exceed 1000 characters'),
  body('questions.*.options').isArray({ min: 2, max: 6 }).withMessage('Each question must have 2-6 options'),
  body('questions.*.correctAnswer').isInt({ min: 0 }).withMessage('Correct answer index is required'),
  body('questions.*.points').optional().isInt({ min: 1 }).withMessage('Points must be a positive number'),
  handleValidationErrors
];

// Pagination validation
export const paginationValidation = [
  query('page').optional().isInt({ min: 1 }).withMessage('Page must be a positive number'),
  query('limit').optional().isInt({ min: 1, max: 100 }).withMessage('Limit must be between 1 and 100'),
  query('sort').optional().isString().withMessage('Sort must be a string'),
  handleValidationErrors
];

// MongoDB ObjectId validation
export const mongoIdValidation = (field: string) => [
  param(field).isMongoId().withMessage(`Valid ${field} is required`),
  handleValidationErrors
];

// src/utils/response.ts
import { Response } from 'express';
import { ApiResponse } from '../types';

export const sendResponse = <T>(
  res: Response,
  statusCode: number,
  success: boolean,
  message: string,
  data?: T,
  pagination?: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  }
): Response => {
  const response: ApiResponse<T> = {
    success,
    message,
    ...(data && { data }),
    ...(pagination && { pagination })
  };

  return res.status(statusCode).json(response);
};

export const sendSuccess = <T>(
  res: Response,
  message: string,
  data?: T,
  statusCode: number = 200
): Response => {
  return sendResponse(res, statusCode, true, message, data);
};

export const sendError = (
  res: Response,
  message: string,
  statusCode: number = 400
): Response => {
  return sendResponse(res, statusCode, false, message);
};

export const sendPaginatedResponse = <T>(
  res: Response,
  message: string,
  data: T[],
  page: number,
  limit: number,
  total: number
): Response => {
  const pages = Math.ceil(total / limit);

  return sendResponse(res, 200, true, message, data, {
    page,
    limit,
    total,
    pages
  });
};

// src/utils/pagination.ts
import { Request } from 'express';
import { Document, Query } from 'mongoose';

export interface PaginationOptions {
  page?: number;
  limit?: number;
  sort?: string;
  select?: string;
  populate?: string | string[];
}

export interface PaginationResult<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  };
}

export const getPaginationOptions = (req: Request): PaginationOptions => {
  const page = parseInt(req.query.page as string) || 1;
  const limit = parseInt(req.query.limit as string) || 10;
  const sort = req.query.sort as string || '-createdAt';
  const select = req.query.select as string;
  const populate = req.query.populate as string;

  return {
    page: Math.max(1, page),
    limit: Math.min(100, Math.max(1, limit)),
    sort,
    select,
    populate: populate ? populate.split(',') : undefined
  };
};

export const applyPagination = async <T extends Document>(
  query: Query<T[], T>,
  options: PaginationOptions
): Promise<PaginationResult<T>> => {
  const { page = 1, limit = 10, sort, select, populate } = options;

  // Count total documents
  const total = await query.model.countDocuments(query.getQuery());

  // Apply pagination, sorting, and selection
  let paginatedQuery = query
    .skip((page - 1) * limit)
    .limit(limit);

  if (sort) {
    paginatedQuery = paginatedQuery.sort(sort);
  }

  if (select) {
    paginatedQuery = paginatedQuery.select(select);
  }

  if (populate) {
    if (Array.isArray(populate)) {
      populate.forEach(field => {
        paginatedQuery = paginatedQuery.populate(field);
      });
    } else {
      paginatedQuery = paginatedQuery.populate(populate);
    }
  }

  const data = await paginatedQuery.exec();

  return {
    data,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit)
    }
  };
};

// src/utils/upload.ts
import { createClient } from '@supabase/supabase-js';
import { config } from '../config/environment';
import multer from 'multer';
import { v4 as uuidv4 } from 'uuid';

// Initialize Supabase client
const supabase = createClient(
  config.SUPABASE_URL!,
  config.SUPABASE_SERVICE_KEY!
);

// Multer configuration for memory storage
const storage = multer.memoryStorage();

export const upload = multer({
  storage,
  limits: {
    fileSize: 100 * 1024 * 1024, // 100MB
  },
  fileFilter: (req, file, cb) => {
    // Allow specific file types
    const allowedMimeTypes = [
      'image/jpeg',
      'image/png',
      'image/gif',
      'video/mp4',
      'video/webm',
      'application/pdf',
      'application/msword',
      'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
      'application/vnd.ms-powerpoint',
      'application/vnd.openxmlformats-officedocument.presentationml.presentation'
    ];

    if (allowedMimeTypes.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('File type not allowed'));
    }
  }
});

export const uploadToSupabase = async (
  file: Express.Multer.File,
  bucket: string,
  folder?: string
): Promise<string> => {
  try {
    const fileName = `${uuidv4()}-${file.originalname}`;
    const filePath = folder ? `${folder}/${fileName}` : fileName;

    const { data, error } = await supabase.storage
      .from(bucket)
      .upload(filePath, file.buffer, {
        contentType: file.mimetype,
        upsert: false
      });

    if (error) {
      throw error;
    }

    // Get public URL
    const { data: publicUrlData } = supabase.storage
      .from(bucket)
      .getPublicUrl(filePath);

    return publicUrlData.publicUrl;
  } catch (error) {
    console.error('Upload error:', error);
    throw new Error('Failed to upload file');
  }
};

export const deleteFromSupabase = async (
  bucket: string,
  filePath: string
): Promise<void> => {
  try {
    const { error } = await supabase.storage
      .from(bucket)
      .remove([filePath]);

    if (error) {
      throw error;
    }
  } catch (error) {
    console.error('Delete error:', error);
    throw new Error('Failed to delete file');
  }
};
```
