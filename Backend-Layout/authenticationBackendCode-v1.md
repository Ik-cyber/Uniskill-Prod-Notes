```js
// src/routes/auth.ts
import express from 'express';
import bcrypt from 'bcryptjs';
import User from '../models/User';
import University from '../models/University';
import Department from '../models/Department';
import { sendSuccess, sendError } from '../utils/response';
import {
  registerStudentValidation,
  registerLecturerValidation,
  loginValidation
} from '../middleware/validation';
import { authenticate } from '../middleware/auth';

const router = express.Router();

// @route   POST /api/auth/register-student
// @desc    Register a new student
// @access  Public
router.post('/register-student', registerStudentValidation, async (req, res) => {
  try {
    const { name, email, password, regNumber, universityId, departmentId, level } = req.body;

    // Check if user already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return sendError(res, 'User with this email already exists', 400);
    }

    // Check if registration number is already used
    const existingRegNumber = await User.findOne({ regNumber, universityId });
    if (existingRegNumber) {
      return sendError(res, 'Registration number already exists for this university', 400);
    }

    // Verify university exists and is approved
    const university = await University.findById(universityId);
    if (!university) {
      return sendError(res, 'University not found', 404);
    }
    if (!university.approved) {
      return sendError(res, 'University is not approved yet', 400);
    }

    // Verify department exists and belongs to university
    const department = await Department.findOne({ _id: departmentId, universityId });
    if (!department) {
      return sendError(res, 'Department not found in this university', 404);
    }

    // Optional: Verify email domain matches university
    const emailDomain = email.split('@')[1];
    if (university.emailSuffix && !email.endsWith(university.emailSuffix)) {
      // Log warning but don't block registration
      console.warn(`Student email domain ${emailDomain} doesn't match university domain ${university.emailSuffix}`);
    }

    // Create new student
    const student = new User({
      name,
      email,
      passwordHash: password, // Will be hashed by pre-save middleware
      role: 'student',
      universityId,
      departmentId,
      regNumber,
      level,
      isVerified: false // Can be verified later by admin or email
    });

    await student.save();

    // Generate auth token
    const token = student.generateAuthToken();

    // Return success response
    sendSuccess(res, 'Student registered successfully', {
      token,
      user: {
        id: student._id,
        name: student.name,
        email: student.email,
        role: student.role,
        universityId: student.universityId,
        departmentId: student.departmentId,
        regNumber: student.regNumber,
        isVerified: student.isVerified
      }
    }, 201);

  } catch (error) {
    console.error('Student registration error:', error);
    sendError(res, 'Registration failed', 500);
  }
});

// @route   POST /api/auth/register-lecturer
// @desc    Register a new lecturer
// @access  Public
router.post('/register-lecturer', registerLecturerValidation, async (req, res) => {
  try {
    const { name, email, password, staffId, universityId, departmentId, specialization } = req.body;

    // Check if user already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return sendError(res, 'User with this email already exists', 400);
    }

    // Check if staff ID is already used
    const existingStaffId = await User.findOne({ staffId, universityId });
    if (existingStaffId) {
      return sendError(res, 'Staff ID already exists for this university', 400);
    }

    // Verify university exists and is approved
    const university = await University.findById(universityId);
    if (!university) {
      return sendError(res, 'University not found', 404);
    }
    if (!university.approved) {
      return sendError(res, 'University is not approved yet', 400);
    }

    // Verify department exists and belongs to university
    const department = await Department.findOne({ _id: departmentId, universityId });
    if (!department) {
      return sendError(res, 'Department not found in this university', 404);
    }

    // Verify email domain matches university (stricter for lecturers)
    const emailDomain = email.split('@')[1];
    if (university.emailSuffix && !email.endsWith(university.emailSuffix)) {
      return sendError(res, `Email must end with ${university.emailSuffix}`, 400);
    }

    // Create new lecturer
    const lecturer = new User({
      name,
      email,
      passwordHash: password, // Will be hashed by pre-save middleware
      role: 'lecturer',
      universityId,
      departmentId,
      staffId,
      specialization,
      isVerified: false // Requires manual verification by admin
    });

    await lecturer.save();

    // Generate auth token
    const token = lecturer.generateAuthToken();

    // Return success response
    sendSuccess(res, 'Lecturer registered successfully. Please wait for admin verification.', {
      token,
      user: {
        id: lecturer._id,
        name: lecturer.name,
        email: lecturer.email,
        role: lecturer.role,
        universityId: lecturer.universityId,
        departmentId: lecturer.departmentId,
        staffId: lecturer.staffId,
        isVerified: lecturer.isVerified
      }
    }, 201);

  } catch (error) {
    console.error('Lecturer registration error:', error);
    sendError(res, 'Registration failed', 500);
  }
});

// @route   POST /api/auth/login
// @desc    Login user
// @access  Public
router.post('/login', loginValidation, async (req, res) => {
  try {
    const { email, password } = req.body;

    // Find user and include password for comparison
    const user = await User.findOne({ email }).select('+passwordHash');
    if (!user) {
      return sendError(res, 'Invalid email or password', 401);
    }

    // Check password
    const isMatch = await user.comparePassword(password);
    if (!isMatch) {
      return sendError(res, 'Invalid email or password', 401);
    }

    // Check if user is verified (optional - can be enforced later)
    if (!user.isVerified && user.role === 'lecturer') {
      return sendError(res, 'Your account is pending verification by admin', 401);
    }

    // Generate auth token
    const token = user.generateAuthToken();

    // Return success response
    sendSuccess(res, 'Login successful', {
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role,
        universityId: user.universityId,
        departmentId: user.departmentId,
        regNumber: user.regNumber,
        staffId: user.staffId,
        isVerified: user.isVerified,
        avatar: user.avatar
      }
    });

  } catch (error) {
    console.error('Login error:', error);
    sendError(res, 'Login failed', 500);
  }
});

// @route   GET /api/auth/me
// @desc    Get current user profile
// @access  Private
router.get('/me', authenticate, async (req: any, res) => {
  try {
    const user = await User.findById(req.user._id)
      .populate('universityId', 'name emailSuffix')
      .
```
