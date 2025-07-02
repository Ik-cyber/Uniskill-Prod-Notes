```js
// src/models/User.ts
import mongoose, { Schema } from 'mongoose';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import { IUser } from '../types';
import { config } from '../config/environment';

const userSchema = new Schema<IUser>({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    maxlength: [100, 'Name cannot exceed 100 characters']
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Please enter a valid email']
  },
  passwordHash: {
    type: String,
    required: [true, 'Password is required'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false
  },
  role: {
    type: String,
    enum: ['student', 'lecturer', 'uni-admin', 'super-admin'],
    required: [true, 'Role is required']
  },
  universityId: {
    type: Schema.Types.ObjectId,
    ref: 'University'
  },
  departmentId: {
    type: Schema.Types.ObjectId,
    ref: 'Department'
  },
  regNumber: {
    type: String,
    sparse: true, // Only for students
    trim: true
  },
  staffId: {
    type: String,
    sparse: true, // Only for lecturers
    trim: true
  },
  isPrivate: {
    type: Boolean,
    default: false
  },
  createdBy: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Creator ID is required']
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Indexes
chatChannelSchema.index({ universityId: 1, type: 1 });
chatChannelSchema.index({ departmentId: 1 });
chatChannelSchema.index({ courseId: 1 });
chatChannelSchema.index({ members: 1 });

export default mongoose.model<IChatChannel>('ChatChannel', chatChannelSchema);

// src/models/ChatMessage.ts
import mongoose, { Schema } from 'mongoose';
import { IChatMessage } from '../types';

const chatMessageSchema = new Schema<IChatMessage>({
  channelId: {
    type: Schema.Types.ObjectId,
    ref: 'ChatChannel',
    required: [true, 'Channel ID is required']
  },
  senderId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Sender ID is required']
  },
  content: {
    type: String,
    required: [true, 'Message content is required'],
    maxlength: [2000, 'Message cannot exceed 2000 characters']
  },
  type: {
    type: String,
    required: [true, 'Message type is required'],
    enum: ['text', 'file', 'image'],
    default: 'text'
  },
  fileUrl: {
    type: String
  },
  replyTo: {
    type: Schema.Types.ObjectId,
    ref: 'ChatMessage'
  },
  edited: {
    type: Boolean,
    default: false
  },
  editedAt: {
    type: Date
  }
}, {
  timestamps: { createdAt: 'sentAt', updatedAt: false }
});

// Indexes
chatMessageSchema.index({ channelId: 1, sentAt: -1 });
chatMessageSchema.index({ senderId: 1 });

export default mongoose.model<IChatMessage>('ChatMessage', chatMessageSchema);Verified: {
    type: Boolean,
    default: false
  },
  avatar: {
    type: String
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ role: 1, universityId: 1 });
userSchema.index({ regNumber: 1 }, { sparse: true });
userSchema.index({ staffId: 1 }, { sparse: true });

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('passwordHash')) return next();

  try {
    const salt = await bcrypt.genSalt(12);
    this.passwordHash = await bcrypt.hash(this.passwordHash, salt);
    next();
  } catch (error) {
    next(error as Error);
  }
});

// Compare password method
userSchema.methods.comparePassword = async function(candidatePassword: string): Promise<boolean> {
  return bcrypt.compare(candidatePassword, this.passwordHash);
};

// Generate JWT token
userSchema.methods.generateAuthToken = function(): string {
  return jwt.sign(
    {
      userId: this._id,
      email: this.email,
      role: this.role,
      universityId: this.universityId,
      departmentId: this.departmentId
    },
    config.JWT_SECRET,
    { expiresIn: config.JWT_EXPIRE }
  );
};

// Virtual for full profile
userSchema.virtual('profile').get(function() {
  return {
    id: this._id,
    name: this.name,
    email: this.email,
    role: this.role,
    isVerified: this.isVerified,
    avatar: this.avatar,
    createdAt: this.createdAt
  };
});

export default mongoose.model<IUser>('User', userSchema);

// src/models/University.ts
import mongoose, { Schema } from 'mongoose';
import { IUniversity } from '../types';

const universitySchema = new Schema<IUniversity>({
  name: {
    type: String,
    required: [true, 'University name is required'],
    unique: true,
    trim: true,
    maxlength: [200, 'University name cannot exceed 200 characters']
  },
  emailSuffix: {
    type: String,
    required: [true, 'Email suffix is required'],
    unique: true,
    lowercase: true,
    match: [/@[\w.-]+\.\w{2,}$/, 'Please enter a valid email suffix (e.g., @unilag.edu.ng)']
  },
  description: {
    type: String,
    maxlength: [1000, 'Description cannot exceed 1000 characters']
  },
  logo: {
    type: String
  },
  approved: {
    type: Boolean,
    default: false
  },
  location: {
    state: {
      type: String,
      required: [true, 'State is required']
    },
    country: {
      type: String,
      default: 'Nigeria'
    }
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Indexes
universitySchema.index({ name: 1 });
universitySchema.index({ emailSuffix: 1 });
universitySchema.index({ approved: 1 });

// Virtual for departments count
universitySchema.virtual('departmentCount', {
  ref: 'Department',
  localField: '_id',
  foreignField: 'universityId',
  count: true
});

export default mongoose.model<IUniversity>('University', universitySchema);

// src/models/Department.ts
import mongoose, { Schema } from 'mongoose';
import { IDepartment } from '../types';

const departmentSchema = new Schema<IDepartment>({
  name: {
    type: String,
    required: [true, 'Department name is required'],
    trim: true,
    maxlength: [100, 'Department name cannot exceed 100 characters']
  },
  code: {
    type: String,
    required: [true, 'Department code is required'],
    uppercase: true,
    trim: true,
    maxlength: [10, 'Department code cannot exceed 10 characters']
  },
  universityId: {
    type: Schema.Types.ObjectId,
    ref: 'University',
    required: [true, 'University ID is required']
  },
  description: {
    type: String,
    maxlength: [500, 'Description cannot exceed 500 characters']
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Compound unique index
departmentSchema.index({ code: 1, universityId: 1 }, { unique: true });
departmentSchema.index({ universityId: 1 });

// Virtual for courses count
departmentSchema.virtual('courseCount', {
  ref: 'Course',
  localField: '_id',
  foreignField: 'departmentId',
  count: true
});

export default mongoose.model<IDepartment>('Department', departmentSchema);

// src/models/Course.ts
import mongoose, { Schema } from 'mongoose';
import { ICourse } from '../types';

const courseSchema = new Schema<ICourse>({
  name: {
    type: String,
    required: [true, 'Course name is required'],
    trim: true,
    maxlength: [200, 'Course name cannot exceed 200 characters']
  },
  code: {
    type: String,
    required: [true, 'Course code is required'],
    uppercase: true,
    trim: true,
    maxlength: [15, 'Course code cannot exceed 15 characters']
  },
  description: {
    type: String,
    required: [true, 'Course description is required'],
    maxlength: [2000, 'Description cannot exceed 2000 characters']
  },
  departmentId: {
    type: Schema.Types.ObjectId,
    ref: 'Department',
    required: [true, 'Department ID is required']
  },
  lecturerId: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  },
  semester: {
    type: Number,
    required: [true, 'Semester is required'],
    enum: [1, 2],
    min: 1,
    max: 2
  },
  level: {
    type: Number,
    required: [true, 'Level is required'],
    enum: [100, 200, 300, 400, 500],
    min: 100,
    max: 500
  },
  credits: {
    type: Number,
    required: [true, 'Credits are required'],
    min: 1,
    max: 6
  },
  isElective: {
    type: Boolean,
    default: false
  },
  prerequisites: [{
    type: Schema.Types.ObjectId,
    ref: 'Course'
  }],
  learningOutcomes: [{
    type: String,
    maxlength: [500, 'Learning outcome cannot exceed 500 characters']
  }]
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Compound unique index
courseSchema.index({ code: 1, departmentId: 1 }, { unique: true });
courseSchema.index({ departmentId: 1 });
courseSchema.index({ lecturerId: 1 });
courseSchema.index({ level: 1, semester: 1 });

// Populate lecturer info
courseSchema.virtual('lecturer', {
  ref: 'User',
  localField: 'lecturerId',
  foreignField: '_id',
  justOne: true
});

export default mongoose.model<ICourse>('Course', courseSchema);

// src/models/Content.ts
import mongoose, { Schema } from 'mongoose';
import { IContent } from '../types';

const contentSchema = new Schema<IContent>({
  courseId: {
    type: Schema.Types.ObjectId,
    ref: 'Course',
    required: [true, 'Course ID is required']
  },
  title: {
    type: String,
    required: [true, 'Content title is required'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters']
  },
  type: {
    type: String,
    required: [true, 'Content type is required'],
    enum: ['video', 'note', 'link', 'assignment']
  },
  content: {
    type: String,
    required: [true, 'Content is required']
  },
  description: {
    type: String,
    maxlength: [1000, 'Description cannot exceed 1000 characters']
  },
  duration: {
    type: Number, // In minutes
    min: 0
  },
  order: {
    type: Number,
    required: [true, 'Content order is required'],
    min: 1
  },
  isPublished: {
    type: Boolean,
    default: false
  },
  uploadedBy: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Uploader ID is required']
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Indexes
contentSchema.index({ courseId: 1, order: 1 });
contentSchema.index({ uploadedBy: 1 });
contentSchema.index({ type: 1 });

export default mongoose.model<IContent>('Content', contentSchema);

// src/models/Quiz.ts
import mongoose, { Schema } from 'mongoose';
import { IQuiz, IQuestion } from '../types';

const questionSchema = new Schema<IQuestion>({
  question: {
    type: String,
    required: [true, 'Question text is required'],
    maxlength: [1000, 'Question cannot exceed 1000 characters']
  },
  options: [{
    type: String,
    required: [true, 'Option text is required'],
    maxlength: [500, 'Option cannot exceed 500 characters']
  }],
  correctAnswer: {
    type: Number,
    required: [true, 'Correct answer index is required'],
    min: 0
  },
  explanation: {
    type: String,
    maxlength: [1000, 'Explanation cannot exceed 1000 characters']
  },
  points: {
    type: Number,
    required: [true, 'Points are required'],
    min: 1,
    default: 1
  }
}, { _id: false });

const quizSchema = new Schema<IQuiz>({
  courseId: {
    type: Schema.Types.ObjectId,
    ref: 'Course',
    required: [true, 'Course ID is required']
  },
  title: {
    type: String,
    required: [true, 'Quiz title is required'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters']
  },
  description: {
    type: String,
    maxlength: [1000, 'Description cannot exceed 1000 characters']
  },
  questions: [questionSchema],
  duration: {
    type: Number,
    required: [true, 'Duration is required'],
    min: 5, // Minimum 5 minutes
    max: 180 // Maximum 3 hours
  },
  passingScore: {
    type: Number,
    required: [true, 'Passing score is required'],
    min: 0,
    max: 100,
    default: 70
  },
  attempts: {
    type: Number,
    required: [true, 'Max attempts is required'],
    min: 1,
    max: 5,
    default: 3
  },
  isPublished: {
    type: Boolean,
    default: false
  },
  createdBy: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Creator ID is required']
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Indexes
quizSchema.index({ courseId: 1 });
quizSchema.index({ createdBy: 1 });

// Virtual for total points
quizSchema.virtual('totalPoints').get(function() {
  return this.questions.reduce((total, question) => total + question.points, 0);
});

export default mongoose.model<IQuiz>('Quiz', quizSchema);

// src/models/QuizSubmission.ts
import mongoose, { Schema } from 'mongoose';
import { IQuizSubmission } from '../types';

const quizSubmissionSchema = new Schema<IQuizSubmission>({
  quizId: {
    type: Schema.Types.ObjectId,
    ref: 'Quiz',
    required: [true, 'Quiz ID is required']
  },
  studentId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Student ID is required']
  },
  answers: [{
    type: Number,
    required: [true, 'Answer is required']
  }],
  score: {
    type: Number,
    required: [true, 'Score is required'],
    min: 0,
    max: 100
  },
  passed: {
    type: Boolean,
    required: [true, 'Pass status is required']
  },
  timeSpent: {
    type: Number,
    required: [true, 'Time spent is required'],
    min: 0
  },
  attemptNumber: {
    type: Number,
    required: [true, 'Attempt number is required'],
    min: 1
  }
}, {
  timestamps: { createdAt: 'submittedAt', updatedAt: false }
});

// Compound indexes
quizSubmissionSchema.index({ quizId: 1, studentId: 1, attemptNumber: 1 }, { unique: true });
quizSubmissionSchema.index({ studentId: 1 });

export default mongoose.model<IQuizSubmission>('QuizSubmission', quizSubmissionSchema);

// src/models/Certificate.ts
import mongoose, { Schema } from 'mongoose';
import { ICertificate } from '../types';

const certificateSchema = new Schema<ICertificate>({
  studentId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Student ID is required']
  },
  courseId: {
    type: Schema.Types.ObjectId,
    ref: 'Course',
    required: [true, 'Course ID is required']
  },
  quizId: {
    type: Schema.Types.ObjectId,
    ref: 'Quiz'
  },
  type: {
    type: String,
    required: [true, 'Certificate type is required'],
    enum: ['course-completion', 'quiz-pass', 'skill-badge']
  },
  title: {
    type: String,
    required: [true, 'Certificate title is required'],
    maxlength: [200, 'Title cannot exceed 200 characters']
  },
  description: {
    type: String,
    required: [true, 'Certificate description is required'],
    maxlength: [1000, 'Description cannot exceed 1000 characters']
  },
  blockchainTxHash: {
    type: String,
    sparse: true
  },
  ipfsHash: {
    type: String,
    sparse: true
  },
  issuedBy: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'Issuer ID is required']
  },
  validUntil: {
    type: Date
  }
}, {
  timestamps: { createdAt: 'issuedAt', updatedAt: false }
});

// Compound unique index
certificateSchema.index({ studentId: 1, courseId: 1, type: 1 }, { unique: true });
certificateSchema.index({ studentId: 1 });
certificateSchema.index({ issuedBy: 1 });

export default mongoose.model<ICertificate>('Certificate', certificateSchema);

// src/models/ChatChannel.ts
import mongoose, { Schema } from 'mongoose';
import { IChatChannel } from '../types';

const chatChannelSchema = new Schema<IChatChannel>({
  name: {
    type: String,
    required: [true, 'Channel name is required'],
    trim: true,
    maxlength: [100, 'Channel name cannot exceed 100 characters']
  },
  type: {
    type: String,
    required: [true, 'Channel type is required'],
    enum: ['department', 'course', 'skill-group']
  },
  universityId: {
    type: Schema.Types.ObjectId,
    ref: 'University',
    required: [true, 'University ID is required']
  },
  departmentId: {
    type: Schema.Types.ObjectId,
    ref: 'Department'
  },
  courseId: {
    type: Schema.Types.ObjectId,
    ref: 'Course'
  },
  description: {
    type: String,
    maxlength: [500, 'Description cannot exceed 500 characters']
  },
  members: [{
    type: Schema.Types.ObjectId,
    ref: 'User'
  }],
  admins: [{
    type: Schema.Types.ObjectId,
    ref: 'User'
  }],
  is
```
