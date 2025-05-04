# AI Literacy Assessment Tool (AILAT) API

## Introduction

The AILAT API provides programmatic access to the AI Literacy Diagnostic Assessment platform, allowing developers to integrate comprehensive AI literacy assessments into their applications, learning management systems, or HR tools.

This REST API enables you to:
- Create and manage assessments
- Retrieve adaptive questions
- Submit user responses (both multiple-choice and open-ended)
- Obtain detailed assessment results with personalized learning recommendations
- Access organization-level analytics

## Authentication

All API requests require authentication using a bearer token in the Authorization header:

```
Authorization: Bearer YOUR_API_TOKEN
```

To obtain an API token, contact our team at [support@ailat.v.ee](mailto:support@ailat.v.ee).

## Base URL

```
https://ailat.v.ee/api/v1
```

## Response Format

All responses are returned in JSON format. Successful responses include a `success: true` field, while error responses include `success: false` and an `error` field.

## Error Handling

The API uses standard HTTP status codes:

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Invalid or missing API token |
| 403 | Forbidden - User lacks permission to access the resource |
| 404 | Not Found - Resource not found |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error |

Error responses follow this format:

```json
{
  "success": false,
  "error": "Error type",
  "message": "Detailed error message",
  "details": "Additional information (optional)",
  "recovery_action": "Suggested action (when applicable)"
}
```

### Error Recovery

When facing service disruptions, the API prioritizes assessment continuity using these approaches:

| Scenario | Error Code | Recovery Mechanism |
|----------|------------|-------------------|
| Question retrieval failure | 500 | Generates fallback questions to allow assessment to continue |
| LLM evaluation unavailable | 200 with note | Falls back to rule-based evaluation (scored 0.5-0.7) with a note in response |
| Assessment data corruption | 500 | Attempts to recover from checkpoints or create a new assessment |
| Rate limit exceeded | 429 | Returns Retry-After header with exponential backoff recommendation |
| AuthorizationError | 403 | Forbidden - User lacks permission to access the resource |

## Rate Limits

The API enforces the following rate limits:
- 100 requests per minute per API token

When rate limits are exceeded, the API returns a 429 status code with a `Retry-After` header indicating the number of seconds to wait before retrying.

## Versioning

The current API version is `v1`. All endpoints described in this documentation are accessible at `/v1/endpoint`.

## Endpoints

### 1. Create Assessment

Create a new assessment for a user.

**Endpoint:** `POST /assessments`

#### Request

```http
POST /v1/assessments
Content-Type: application/json
Authorization: Bearer YOUR_API_TOKEN

{
  "industry": "Healthcare & Social Services",
  "role": "Physician",
  "motivation": "Professional Development",
  "organization_id": "org_abc123",
  "user_id": "user_xyz789"
}
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| industry | string (enum) | Yes | User's industry - must be one of the predefined industry categories (see Industry Categories section) |
| role | string | Yes | User's professional role |
| motivation | string (enum) | No | User's reason for taking the assessment (see Motivation Categories section) |
| organization_id | string | No | Organization identifier for tracking purposes |
| user_id | string | No | User identifier for tracking purposes |

#### Industry Categories

The following industry categories are supported:

| Value | Description |
|-------|-------------|
| Healthcare & Social Services | Healthcare providers, social services, etc. |
| Financial Services | Banking, insurance, investment firms |
| Information Technology | Software development, IT services |
| Manufacturing | Industrial manufacturing, production |
| Education | Schools, universities, training institutions |
| Retail & E-commerce | Retail stores, online commerce |
| Government & Public Sector | Government agencies, public services |
| Transportation & Logistics | Shipping, transport, supply chain |
| Arts, Entertainment & Media | Media companies, entertainment |
| Energy & Utilities | Power, water, gas utilities |
| Construction & Real Estate | Building, property management |
| Professional Services | Legal, consulting, accounting |
| Telecommunications | Telecom providers, networks |
| Agriculture, Forestry & Fishing | Farming, forestry industries |
| Hospitality & Tourism | Hotels, restaurants, tourism |
| Wholesale & Distribution | Wholesale trade, distribution |
| General (Cross-industry) | Not industry-specific |

#### Motivation Categories

*Note: This field is optional and accepts free-form text, but common values include:*

- Professional Development
- Compliance/Requirement
- Personal Interest
- Skill Assessment
- Career Advancement

#### Response

```json
{
  "success": true,
  "assessment_id": "asmt_abcdef123456",
  "status": "in_progress",
  "current_question": {
    "question_id": "Q1",
    "question_text": "Which of the following best describes what 'machine learning' is?",
    "question_type": "multiple_choice",
    "options": {
      "A": "A machine that is capable of learning on its own without any data or human input.",
      "B": "A hardware device that stores all human knowledge for a computer to use.",
      "C": "A method that allows computers to learn from data and experience, rather than being explicitly programmed for each task.",
      "D": "A robotic machine that mimics human learning by physically observing and copying people."
    },
    "dimension": "CONCEPTUAL_KNOWLEDGE"
  },
  "progress": {
    "questions_completed": 0,
    "total_questions": 20,
    "percent_complete": 0
  }
}
```

### 2. Get Current Assessment State

Retrieve the current state of an assessment, including the current question and progress.

**Endpoint:** `GET /assessments/{assessmentId}`

#### Request

```http
GET /v1/assessments/asmt_abcdef123456
Authorization: Bearer YOUR_API_TOKEN
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| assessmentId | string | Yes | Assessment identifier returned from the create assessment endpoint |

#### Response

```json
{
  "success": true,
  "assessment_id": "asmt_abcdef123456",
  "status": "in_progress",
  "current_question": {
    "question_id": "Q1",
    "question_text": "Which of the following best describes what 'machine learning' is?",
    "question_type": "multiple_choice",
    "options": {
      "A": "A machine that is capable of learning on its own without any data or human input.",
      "B": "A hardware device that stores all human knowledge for a computer to use.",
      "C": "A method that allows computers to learn from data and experience, rather than being explicitly programmed for each task.",
      "D": "A robotic machine that mimics human learning by physically observing and copying people."
    },
    "dimension": "CONCEPTUAL_KNOWLEDGE",
    "context": null
  },
  "progress": {
    "questions_completed": 0,
    "total_questions": 20,
    "percent_complete": 0
  }
}
```

### 3. Submit Answer

Submit an answer to a question (works for both multiple-choice and open-ended questions).

**Endpoint:** `POST /assessments/{assessmentId}/answers`

#### Request for Multiple-Choice

```http
POST /v1/assessments/asmt_abcdef123456/answers
Content-Type: application/json
Authorization: Bearer YOUR_API_TOKEN

{
  "question_id": "Q1",
  "answer": "C",
  "response_time_ms": 45000
}
```

#### Request for Open-Ended

```http
POST /v1/assessments/asmt_abcdef123456/answers
Content-Type: application/json
Authorization: Bearer YOUR_API_TOKEN

{
  "question_id": "OE1",
  "answer": "AI hallucinations are a phenomenon where AI models, especially large language models, generate content that appears factual but is actually incorrect or fabricated. This happens because these models predict likely text patterns based on their training data rather than retrieving verified facts. When using AI systems that might hallucinate, it's important to verify any factual claims with trusted sources, especially for important or specialized information. Understanding the limitations of these models helps users implement appropriate verification steps and maintain realistic expectations about AI capabilities.",
  "response_time_ms": 180000
}
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| assessmentId | string | Yes | Assessment identifier |

**Request Body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| question_id | string | Yes | ID of the question being answered |
| answer | string | Yes | Selected option (A, B, C, or D) for multiple-choice or text response for open-ended |
| response_time_ms | number | No | Time in milliseconds spent answering the question |

#### Response for Multiple-Choice

```json
{
  "success": true,
  "is_correct": false,
  "correct_answer": "C",
  "explanation": "Incorrect. Machine learning is a method that allows computers to learn from data and experience, rather than being explicitly programmed for each task. Option A incorrectly suggests machines can learn without data. Option B describes a database, not machine learning. Option D incorrectly characterizes ML as physical imitation.",
  "next_question": {
    "question_id": "Q2",
    "question_text": "What is a hallucination in the context of large language models?",
    "question_type": "multiple_choice",
    "options": {
      "A": "When an AI model becomes sentient and creates its own reality.",
      "B": "When an AI generates content that appears factual but is inaccurate or made up.",
      "C": "When an AI model visualizes images internally before generating text.",
      "D": "When an AI model crashes due to contradictory instructions."
    },
    "dimension": "CONCEPTUAL_KNOWLEDGE",
    "context": null
  },
  "progress": {
    "questions_completed": 1,
    "total_questions": 20,
    "percent_complete": 5
  },
  "assessment_complete": false
}
```

#### Response for Open-Ended

```json
{
  "success": true,
  "evaluation": {
    "overall_score": 0.85,
    "feedback": "Your explanation demonstrates excellent conceptual understanding of AI hallucinations. Your definition is accurate and you correctly identify that hallucinations stem from statistical pattern prediction rather than fact retrieval. Your application advice about verification is sound, though you could strengthen it with specific verification techniques. Consider expanding on how hallucinations might manifest differently across various AI applications or contexts."
  },
  "next_question": {
    "question_id": "Q8",
    "question_text": "What is the most appropriate approach when implementing AI tools in a workflow?",
    "question_type": "multiple_choice",
    "options": {
      "A": "Using the most advanced AI available regardless of the specific task",
      "B": "Fully automating all processes to eliminate human error",
      "C": "Determining where AI adds value and where human judgment is needed",
      "D": "Ensuring AI makes all final decisions for consistency"
    },
    "dimension": "USE_APPLY_KNOWLEDGE",
    "context": null
  },
  "progress": {
    "questions_completed": 7,
    "total_questions": 20,
    "percent_complete": 35
  },
  "assessment_complete": false
}
```

When an assessment is complete, the response will include a reference to get results:

```json
{
  "success": true,
  "is_correct": true,
  "explanation": "Correct! The explanation for your answer...",
  "correct_answer": "C",
  "assessment_complete": true,
  "results_url": "/v1/assessments/asmt_abcdef123456/results",
  "progress": {
    "questions_completed": 20,
    "total_questions": 20,
    "percent_complete": 100
  }
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| success | boolean | Indicates if the request was successful |
| is_correct | boolean | For multiple-choice questions, whether the answer was correct |
| correct_answer | string | For multiple-choice questions, the correct option key (A, B, C, or D) |
| explanation | string | For multiple-choice questions, explanation of the correct answer |
| evaluation | object | For open-ended questions, contains overall_score and feedback |
| next_question | object | The next question to be answered |
| assessment_complete | boolean | Whether the assessment is complete |
| results_url | string | URL to access results if assessment is complete |
| progress | object | Information about assessment progress |

### 4. Update Assessment Status

Pause or resume an assessment.

**Endpoint:** `PUT /assessments/{assessmentId}/status`

#### Request

```http
PUT /v1/assessments/asmt_abcdef123456/status
Content-Type: application/json
Authorization: Bearer YOUR_API_TOKEN

{
  "status": "paused"
}
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| assessmentId | string | Yes | Assessment identifier |

**Request Body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| status | string (enum) | Yes | New status. Must be either "paused" or "in_progress" |

#### Response

```json
{
  "success": true,
  "assessment_id": "asmt_abcdef123456",
  "status": "paused",
  "resume_url": "/v1/assessments/asmt_abcdef123456",
  "expires_at": "2025-04-07T14:30:45Z"
}
```

### 5. Get Assessment Results

Retrieve the results of a completed assessment.

**Endpoint:** `GET /assessments/{assessmentId}/results`

#### Request

```http
GET /v1/assessments/asmt_abcdef123456/results
Authorization: Bearer YOUR_API_TOKEN
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| assessmentId | string | Yes | Assessment identifier |

#### Response

```json
{
  "success": true,
  "assessment_id": "asmt_abcdef123456",
  "user_id": "user_xyz789",
  "organization_id": "org_abc123",
  "completed_at": "2025-04-05T15:15:12Z",
  "duration_minutes": 45,
  "overall_score": 76.5,
  "ai_literacy_level": 4,
  "level_details": {
    "title": "Critical Evaluator",
    "description": "You demonstrate strong proficiency in analyzing and utilizing AI systems. You understand core concepts, can effectively use AI tools, critically evaluate outputs, and consider ethical implications.",
    "indicators": [
      "Regularly applies AI tools to solve complex problems",
      "Can critically evaluate AI outputs",
      "Understands technical limitations of AI systems",
      "Considers ethical implications of AI use"
    ]
  },
  "dimension_scores": {
    "CONCEPTUAL_KNOWLEDGE": 85.0,
    "USE_APPLY_KNOWLEDGE": 75.0,
    "EVALUATE_CREATE_KNOWLEDGE": 70.0,
    "ETHICS_KNOWLEDGE": 65.0
  },
  "strengths": [
    "Strong understanding of AI concepts and terminology",
    "Effective at practical application of AI tools"
  ],
  "growth_areas": [
    "Enhance awareness of AI ethics and responsible use practices",
    "Develop deeper skills in evaluating AI outputs"
  ],
  "learning_path": {
    "focus_areas": [
      "AI ethics in healthcare contexts",
      "Critical evaluation techniques for AI-generated content",
      "Responsible AI implementation frameworks"
    ],
    "resources": [
      {
        "title": "AI Ethics: Principles for Healthcare Professionals",
        "type": "course",
        "url": "https://example.com/ai-ethics-healthcare",
        "relevance": "Addresses ethical considerations specific to healthcare AI applications"
      },
      {
        "title": "Evaluating AI Outputs: A Practical Guide",
        "type": "ebook",
        "url": "https://example.com/evaluating-ai-guide",
        "relevance": "Provides frameworks for critically assessing AI-generated content"
      },
      {
        "title": "Responsible AI Implementation Toolkit",
        "type": "toolkit",
        "url": "https://example.com/responsible-ai-toolkit",
        "relevance": "Offers step-by-step guidance for implementing AI responsibly in clinical settings"
      }
    ],
    "next_steps": [
      "Complete the AI Ethics course to strengthen your understanding of responsible AI practices",
      "Apply the evaluation framework to assess outputs from AI tools you currently use",
      "Develop a checklist for ethical considerations when implementing new AI systems"
    ]
  }
}
```

### 6. Get Organization Results

Retrieve aggregated assessment results for an organization.

**Endpoint:** `GET /organizations/{organizationId}/results`

#### Request

```http
GET /v1/organizations/org_abc123/results
Authorization: Bearer YOUR_API_TOKEN
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| organizationId | string | Yes | Organization identifier |

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| time_period | string | No | `all_time` | Filter results by time period. Valid values: `all_time`, `last_7_days`, `last_30_days`, `last_90_days`, `current_year` |
| limit | number | No | 100 | Maximum number of results to return (1-500) |
| cursor | string | No | null | Pagination cursor for fetching next page |

#### Response

```json
{
  "success": true,
  "organization_id": "org_abc123",
  "pagination": {
    "limit": 100,
    "next_cursor": "base64_encoded_assessment_id"
  },
  "total_assessments": 25,
  "completed_assessments": 23,
  "average_level": 3.6,
  "average_score": 72.5,
  "dimension_averages": {
    "CONCEPTUAL_KNOWLEDGE": 78.2,
    "USE_APPLY_KNOWLEDGE": 75.7,
    "EVALUATE_CREATE_KNOWLEDGE": 70.1,
    "ETHICS_KNOWLEDGE": 65.4
  },
  "strongest_dimension": "CONCEPTUAL_KNOWLEDGE",
  "weakest_dimension": "ETHICS_KNOWLEDGE",
  "time_period": {
    "start_time": "2025-01-01T00:00:00Z",
    "end_time": "2025-04-24T23:59:59Z"
  },
  "completion_rate": 0.92,
  "level_distribution": {
    "1": 0,
    "2": 3,
    "3": 8,
    "4": 10,
    "5": 2
  },
  "role_analytics": {
    "Manager": {
      "count": 6,
      "completed": 5,
      "completion_rate": 0.83,
      "average_score": 74.2,
      "average_level": 3.8,
      "dimension_averages": {
        "CONCEPTUAL_KNOWLEDGE": 76.5,
        "USE_APPLY_KNOWLEDGE": 72.8,
        "EVALUATE_CREATE_KNOWLEDGE": 68.9,
        "ETHICS_KNOWLEDGE": 62.6
      }
    },
    "Developer": {
      "count": 8,
      "completed": 8,
      "completion_rate": 1.0,
      "average_score": 76.8,
      "average_level": 4.1,
      "dimension_averages": {
        "CONCEPTUAL_KNOWLEDGE": 80.2,
        "USE_APPLY_KNOWLEDGE": 79.3,
        "EVALUATE_CREATE_KNOWLEDGE": 72.5,
        "ETHICS_KNOWLEDGE": 68.0
      }
    }
  },
  "department_analytics": {
    "Engineering": {
      "count": 10,
      "completed": 9,
      "completion_rate": 0.9,
      "average_score": 75.3,
      "average_level": 3.9,
      "dimension_averages": {
        "CONCEPTUAL_KNOWLEDGE": 79.5,
        "USE_APPLY_KNOWLEDGE": 76.1,
        "EVALUATE_CREATE_KNOWLEDGE": 71.2,
        "ETHICS_KNOWLEDGE": 65.8
      }
    }
  },
  "time_analytics": {
    "monthly_data": {
      "2025-03": {
        "count": 10,
        "avg_score": 70.5,
        "avg_level": 3.4,
        "dimension_averages": {
          "CONCEPTUAL_KNOWLEDGE": 75.5,
          "USE_APPLY_KNOWLEDGE": 72.0,
          "EVALUATE_CREATE_KNOWLEDGE": 68.2,
          "ETHICS_KNOWLEDGE": 63.5
        }
      },
      "2025-04": {
        "count": 13,
        "avg_score": 74.2,
        "avg_level": 3.8,
        "dimension_averages": {
          "CONCEPTUAL_KNOWLEDGE": 79.8,
          "USE_APPLY_KNOWLEDGE": 77.6,
          "EVALUATE_CREATE_KNOWLEDGE": 72.3,
          "ETHICS_KNOWLEDGE": 67.2
        }
      }
    },
    "weekly_data": [
      {
        "week": "2025-04-W1",
        "count": 5,
        "average_score": 73.2
      }
      // ... more weekly data ...
    ],
    "trend_analysis": [
      {
        "period": "2025-04",
        "assessment_count": 13,
        "score_change": 3.7,
        "level_change": 0.4
      }
    ]
  },
  "training_recommendations": {
    "organization_initiatives": [
      {
        "title": "AI Literacy Baseline Program",
        "description": "Organization-wide training to establish common understanding of AI concepts, capabilities, and limitations.",
        "target_audience": "All employees",
        "timeline": "3 months"
      },
      {
        "title": "Role-Specific AI Training Pathways",
        "description": "Tailored learning paths for different roles based on their AI literacy needs.",
        "target_audience": "All employees",
        "timeline": "6 months"
      }
    ],
    "dimension_specific_workshops": [
      {
        "title": "Responsible AI Guidelines",
        "description": "Develop and train employees on ethical AI usage guidelines.",
        "target_audience": "All employees",
        "duration": "1 session, 3 hours + reinforcement"
      },
      {
        "title": "AI Ethics Committee",
        "description": "Form a cross-functional team to review AI implementations.",
        "target_audience": "Representatives from all departments",
        "duration": "Ongoing, monthly meetings"
      }
    ],
    "implementation_plan": {
      "month_1": "Baseline assessment completion and analysis",
      "month_2": "Initial training sessions focused on weakest dimension",
      "month_3": "Role-specific training pathways launch"
    }
  },
  "recent_assessments": [
    {
      "assessment_id": "asmt_abcdef123456",
      "user_id": "user_xyz789",
      "completed_at": "2025-04-05T14:30:45Z",
      "overall_score": 76.5,
      "ai_literacy_level": 4
    },
    {
      "assessment_id": "asmt_456abc",
      "user_id": "user_abc456",
      "completed_at": "2025-04-04T10:15:22Z",
      "overall_score": 68.3,
      "ai_literacy_level": 3
    }
  ]
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| organization_id | string | Organization identifier |
| pagination | object | Pagination information for result set |
| total_assessments | number | Total number of assessments taken in the organization |
| completed_assessments | number | Number of assessments that were completed |
| average_level | number | Average AI literacy level across all completed assessments |
| average_score | number | Average overall score across all completed assessments |
| dimension_averages | object | Average scores for each literacy dimension |
| strongest_dimension | string | Dimension where the organization scores highest |
| weakest_dimension | string | Dimension where the organization scores lowest |
| time_period | object | Time range for the analysis with start_time and end_time in ISO format |
| completion_rate | number | Ratio of completed to total assessments |
| level_distribution | object | Distribution of assessments by literacy level |
| role_analytics | object | Analytics broken down by role |
| department_analytics | object | Analytics broken down by department |
| time_analytics | object | Analytics over time, including both monthly_data and weekly_data |
| training_recommendations | object | Recommended training initiatives based on assessment results |
| recent_assessments | array | List of recently completed assessments |

### 6. Get Assessment Status

Get the status of an assessment without retrieving the current question.

**Endpoint:** `GET /assessments/{assessmentId}/status`

#### Request

```http
GET /v1/assessments/asmt_abcdef123456/status
Authorization: Bearer YOUR_API_TOKEN
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| assessmentId | string | Yes | Assessment identifier |

#### Response

```json
{
  "success": true,
  "assessment_id": "asmt_abcdef123456",
  "status": "in_progress",
  "user_id": "user_xyz789",
  "organization_id": "org_abc123",
  "created_at": "2025-04-24T14:30:45Z",
  "last_activity": "2025-04-24T14:35:12Z",
  "progress": {
    "questions_completed": 7,
    "total_questions": 20,
    "percent_complete": 35
  },
  "expires_at": "2025-04-25T14:30:45Z"
}
```

## AI Literacy Dimensions

The assessment measures four key dimensions of AI literacy:

### 1. CONCEPTUAL_KNOWLEDGE
Understanding of AI fundamentals, terminology, capabilities, and limitations. This includes knowledge of how different AI systems function, their components, and underlying principles.

### 2. USE_APPLY_KNOWLEDGE
Practical skills in effectively using AI tools. This encompasses prompt engineering, implementation strategies, and integration with existing workflows.

### 3. EVALUATE_CREATE_KNOWLEDGE
Ability to critically assess AI outputs for quality, accuracy, and limitations, and to develop new AI-enhanced solutions.

### 4. ETHICS_KNOWLEDGE
Understanding of responsible AI, including issues of bias, privacy, transparency, and accountability.

## AI Literacy Levels

Assessment results classify users into five AI literacy levels:

### Level 1: Baseline Awareness (Score 0-20)
- Basic awareness of AI concepts but limited practical knowledge
- Can identify what AI is in general terms
- Limited understanding of how AI works
- Minimal experience using AI tools

### Level 2: Informed User (Score 21-40)
- Familiar with AI concepts and occasional use of AI tools
- Recognizes common AI applications
- Uses basic AI tools with guidance
- Aware of some limitations of AI

### Level 3: Capable Practitioner (Score 41-60)
- Competent understanding of AI and regular use of AI tools
- Effectively uses AI for routine tasks
- Understands core AI concepts
- Can identify potential biases in AI systems
- Aware of ethical considerations

### Level 4: Critical Evaluator (Score 61-80)
- Proficient understanding and skilled application of AI
- Regularly applies AI tools to solve complex problems
- Can critically evaluate AI outputs
- Understands technical limitations of AI systems
- Considers ethical implications of AI use

### Level 5: AI Champion (Score 81-100)
- Advanced understanding and sophisticated application of AI
- Deep understanding of AI capabilities and limitations
- Expert at using AI tools creatively and effectively
- Can evaluate and mitigate biases in AI systems
- Strong grasp of ethical and societal implications

## Assessment Structure

The assessment follows a three-phase structure:

### 1. Calibration Phase (5 questions)
- Initial questions to establish baseline knowledge
- One question from each dimension plus one additional
- Establishes baseline proficiency

### 2. Adaptive Phase (10 questions) 
- 8 multiple-choice questions
- 2 open-ended questions
- Dynamically adjusts to user performance

### 3. Scenario Phase (5 questions)
- Industry-specific scenario
- 4 multiple-choice questions
- 1 open-ended capstone question

The assessment dynamically adjusts question difficulty and focus areas based on user performance.

## Best Practices

1. **Session Management**
   - Store the assessment ID securely between requests
   - Sessions expire after 24 hours (configurable via SESSION_TTL)
   - Handle session timeouts gracefully by creating new assessments if needed
   - Check the `expires_at` field in status responses to anticipate expiration

2. **Response Time Tracking**
   - Include response_time_ms in answer submissions when possible
   - This helps improve the assessment's accuracy through IRT calculations
   - Response times are analyzed for performance insights
   - Reasonable ranges: 5-60 seconds for multiple-choice, 60-300 seconds for open-ended

3. **Error Handling**
   - Implement retry logic for rate limiting (429 errors)
   - Use the `recovery_action` field for user-friendly error messages
   - Follow exponential backoff with jitter for retries
   - Handle different error types appropriately (400, 401, 403, 404, 429, 500)
   - Log error details for debugging but display sanitized messages to users

4. **Progress Tracking**
   - Use the `progress` object in responses to update UI
   - Consider showing phase transitions (Calibration → Adaptive → Scenario)
   - Update users when assessment_complete is true
   - Handle both percentage and question count displays

5. **Result Visualization**
   - Create radar charts for dimension scores visualization
   - Use color coding for AI literacy levels (1-5)
   - Highlight strengths (scores ≥70%) and growth areas (scores <60%)
   - Consider progress indicators for learning path completion

6. **Learning Integration**
   - Link directly to recommended resources in learning_path
   - Track resource engagement for analytics
   - Consider integrating with your LMS via organization_id
   - Implement learning path progress tracking

7. **Organization Analytics**
   - Use pagination for large result sets (limit and cursor)
   - Cache frequently requested time periods
   - Consider role-based access control for analytics
   - Implement data export functionality for offline analysis

## Advanced Usage

### Fallback Handling
The API includes automatic fallback mechanisms:
- Question generation when bank is exhausted
- Rule-based evaluation when LLM is unavailable
- Session recovery from checkpoints
- Graceful degradation for non-critical features

### Custom Integration Examples

```javascript
// Example: Retry logic with exponential backoff
async function apiRequestWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After') || 5;
        await new Promise(resolve => 
          setTimeout(resolve, (retryAfter * 1000) * Math.pow(2, i) + Math.random() * 1000)
        );
        continue;
      }
      return response;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
    }
  }
}

// Example: Session recovery
async function getOrCreateAssessment(existingId, userData) {
  if (existingId) {
    try {
      const response = await fetch(`/api/v1/assessments/${existingId}/status`);
      if (response.ok) return existingId;
    } catch (error) {
      console.warn('Session expired, creating new assessment');
    }
  }
  return createNewAssessment(userData);
}
```

## Support

For questions, issues, or feature requests, please contact:
- Email: support@ailat.v.ee
- API Status: https://ailat.v.ee/api/v1/health

## Changelog

### v1.0.0 (2025-04-27)
- Initial API release
- Core assessment endpoints
- Organization analytics
- Adaptive testing implementation
- Learning path recommendations

### Upcoming Features (v1.1)
- Batch assessments
- Custom question bank integration
- Webhook notifications
- Enhanced organization management
- Data export capabilities
