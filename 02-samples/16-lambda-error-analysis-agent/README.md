# AI-Powered Lambda Error Analysis with Strands Agents

[![AWS CDK](https://img.shields.io/badge/AWS-CDK-orange)](https://aws.amazon.com/cdk/)
[![Python](https://img.shields.io/badge/Python-3.12-blue)](https://www.python.org/)
[![Strands Agents](https://img.shields.io/badge/Strands-Agents-green)](https://strandsagents.com/)
[![Amazon Bedrock](https://img.shields.io/badge/Amazon-Bedrock-purple)](https://aws.amazon.com/bedrock/)

> Intelligent error analysis agent that automatically investigates Lambda failures using source code, CloudWatch logs, and knowledge base to deliver root cause identification, specific fix recommendations, and confidence scoring

Transform generic Lambda error messages into intelligent, actionable insights using AI-powered analysis. Instead of cryptic stack traces, get root cause explanations, specific code fixes, and confidence-scored recommendations.

## What This Does

When a Lambda function fails, instead of getting generic errors like:
```
KeyError: 'email'
AttributeError: 'NoneType' object has no attribute 'lower'
ZeroDivisionError: division by zero
```

You get intelligent AI analysis like:
```
🤖 Root Cause Analysis:
The function expects user_data to have an 'email' field, but the incoming 
data only contains {"profile":{"name":"John Doe"},"age":30} - no 'email' key.

💡 Immediate Fix:
Add input validation before processing:

def enrich_user_profile(user_data):
    # Validate required fields
    required_fields = ['email', 'profile', 'age']
    missing_fields = [f for f in required_fields if f not in user_data]
    
    if missing_fields:
        raise ValueError(f"Missing required fields: {missing_fields}")
    
    email = user_data['email'].lower().strip()
    # ... rest of function

📊 Confidence: 0.85 (High)
📈 Evidence: Source code retrieved, execution logs analyzed, 3 KB docs found
```

## Why This Matters

**Traditional Lambda debugging:**
- ❌ Generic error messages with no context
- ❌ Manual log diving across CloudWatch
- ❌ Time-consuming root cause analysis
- ❌ Repetitive troubleshooting for similar errors

**With AI-Powered Analysis:**
- ✅ Instant root cause identification
- ✅ Specific, actionable fix recommendations
- ✅ Confidence-scored analysis with evidence
- ✅ Automated investigation across multiple sources
- ✅ Historical pattern recognition

## Architecture

![Lambda Error Analysis Architecture](diagrams/lambda-error-analysis-architecture.png)

> 💡 **Key Innovation**: The agent doesn't just read logs - it combines source code analysis, execution logs, and documentation to provide context-aware recommendations that understand your specific implementation.

### How It Works

1. **Business Lambda Fails** → Decorated with `@error_capture`
2. **Error Event Published** → EventBridge receives failure details
3. **AI Agent Triggered** → Strands Agent analyzes the error using:
   - **Source Code Analysis** - Fetches actual Lambda code from S3/deployment package
   - **CloudWatch Logs** - Retrieves execution logs with request ID filtering
   - **Knowledge Base** - Searches documentation and error patterns
4. **Enhanced Analysis Returned** → Root cause + actionable fixes + confidence score
5. **Results Stored** → DynamoDB for historical analysis and improvement

### Key Components

**Sample Business Function** (`sample-business-function/`)
- Simulates a digital banking user registration microservice
- Validates customer data, enriches profiles, assigns membership tiers
- Contains 8 intentional validation bugs for demonstration:
  - Missing required fields (email, profile)
  - Null value handling
  - Type conversion errors (age, dates)
  - Division by zero (edge cases)
  - Nested data access failures
  - Array index errors (name parsing)

**Error Analyzer Agent** (`error-analyzer-agent/`)
- Strands Agent with Claude Sonnet 4 featuring **interleaved thinking**
  - Reasons between tool calls for smarter investigation
  - Chains multiple tools with reasoning steps in between
  - Makes nuanced decisions based on intermediate results
- Three specialized tools:
  - `fetch_source_code` - Retrieves Lambda source from S3 or deployment package
  - `fetch_cloudwatch_logs` - Gets execution logs filtered by request ID
  - `search_knowledge_base` - Queries Amazon Bedrock Knowledge Base
- Confidence scoring system (0.0-1.0) based on evidence quality
- Stores analysis results in DynamoDB with full context

**@error_capture Decorator** (`decorator.py`)
- Wraps business functions to catch all exceptions
- Publishes structured error events to EventBridge
- Includes stack traces, context, and CloudWatch links
- Security control via `expose_errors` parameter

## Live Demo

Watch the agent analyze a real Lambda error:

```python
# Error: KeyError: 'email'
# Input: {"profile": {"name": "Jane"}, "age": 25}

# AI Analysis (in 8 seconds):
✅ Root Cause: Missing 'email' field in user_data
✅ Fix: Add input validation before processing
✅ Confidence: 0.85 (High)
✅ Evidence: Source code + logs + 3 KB docs
```

## Quick Start

### Prerequisites

- **AWS Account** with appropriate permissions
- **AWS CLI** installed and configured (`aws configure`)
- **Node.js** 18+ and npm
- **Python** 3.12+
- **Amazon Bedrock** access with Claude models enabled

### 1. Clone and Install

```bash
cd lambda-error-analysis-agent
npm install
```

### 2. Deploy Infrastructure

Open the Jupyter notebook for interactive deployment:

```bash
jupyter notebook deploy-agent.ipynb
```

The notebook will:
- ✅ Deploy CDK stack (Lambda functions, EventBridge, DynamoDB, Knowledge Base)
- ✅ Discover deployed function names
- ✅ Run 9 test scenarios with different validation errors
- ✅ Display AI analysis results with confidence scores
- ✅ Show analysis history from DynamoDB

**Or deploy via CDK directly:**

```bash
npm run cdk deploy
```

### 3. Test the System

The notebook includes 9 pre-configured test scenarios:

```python
# Test 1: Missing Email Field
test_payload_missing_email = {
    "user_data": {
        "profile": {"name": "Jane Smith"},
        "age": 25
    }
}

# Invoke and wait for AI analysis
response = lambda_client.invoke(
    FunctionName=business_function_name,
    Payload=json.dumps(test_payload_missing_email)
)

request_id = response['ResponseMetadata']['RequestId']
wait_for_analyzer_logs(analyzer_function_name, request_id)
get_analyzer_logs(analyzer_function_name, request_id)
```

## Project Structure

```
lambda-error-analysis-agent/
├── cdk/
│   ├── lambda/
│   │   ├── sample-business-function/      # Business logic with @error_capture
│   │   │   ├── lambda_function.py         # User registration processor
│   │   │   ├── decorator.py               # @error_capture decorator
│   │   │   └── test-events.json           # 9 test scenarios
│   │   └── error-analyzer-agent/          # AI Error Analyzer
│   │       ├── lambda_function.py         # Main handler
│   │       ├── agent.py                   # Strands Agent with 3 tools
│   │       └── requirements.txt
│   ├── layers/strands-layer/              # Lambda Layer with dependencies
│   └── stacks/lambda-error-analysis-stack.ts
├── knowledge_base/                        # Documentation for AI agent
│   ├── common-errors.md
│   ├── lambda-error-patterns.md
│   ├── user-data-validation-patterns.md
│   └── troubleshooting-guide.md
├── diagrams/                              # Architecture diagrams
│   ├── generate_diagram.py               # Reproducible diagram generation
│   └── lambda-error-analysis-architecture.png
├── deploy-agent.ipynb                     # Interactive deployment notebook
├── cdk-app.ts                            # CDK entry point
├── package.json                          # CDK dependencies
└── README.md                             # This file - comprehensive documentation
```

## Sample Business Function

The sample function simulates a **digital banking user registration service** that:

1. **Processes new customer signups** - Validates email, name, age, initial deposit
2. **Enriches user profiles** - Normalizes data, calculates 10% signup bonus
3. **Assigns membership tiers** - Bronze/Silver/Gold/Platinum based on:
   - Balance-to-age ratio (70% weight) - measures customer lifetime value
   - Notification preferences (30% weight) - measures engagement potential

**Tier Calculation:**
```python
balance_per_year = balance / age
tier_score = (balance_per_year × 0.7) + (notification_count × 0.3)

if tier_score > 1000: return 'platinum'
elif tier_score > 500: return 'gold'
elif tier_score > 100: return 'silver'
else: return 'bronze'
```

This example includes typical validation scenarios that can occur in user onboarding systems, providing a practical testbed for demonstrating AI-powered error analysis.

## Test Scenarios

The system includes 9 comprehensive test cases covering common validation errors:

| Test | Error Type | Trigger | Expected Analysis |
|------|-----------|---------|-------------------|
| 1 | `KeyError: 'email'` | Missing email field | Identifies missing required field, suggests validation |
| 2 | `AttributeError` | Null email value | Detects null handling issue, recommends safe access |
| 3 | `KeyError: 'profile'` | Missing profile object | Points to missing nested data structure |
| 4 | `IndexError` | Single name (no last name) | Identifies array access error in name parsing |
| 5 | `ValueError` | Invalid age type | Catches type conversion failure |
| 6 | `ZeroDivisionError` | Age = 0 | Detects edge case in tier calculation |
| 7 | `ValueError` | Invalid date format | Identifies date parsing mismatch |
| 8 | `KeyError` | Missing nested settings | Finds missing nested dictionary access |
| 9 | Valid | Successful processing | Confirms system works with valid data |

## AI Agent Features

### Confidence Scoring System

The agent calculates confidence based on evidence quality:

```python
Confidence Score = Knowledge Base (0.0-0.4) + 
                   Source Code (0.0-0.3) + 
                   CloudWatch Logs (0.0-0.3)

Levels:
- 0.8+ : Very High (all evidence available, high KB relevance)
- 0.6-0.8 : High (most evidence available)
- 0.4-0.6 : Medium (partial evidence)
- 0.2-0.4 : Low (limited evidence)
- <0.2 : Very Low (minimal evidence)
```

### Interleaved Thinking with Claude Sonnet 4

The agent uses **Claude Sonnet 4 with interleaved thinking** - a powerful capability that enables the AI to reason between tool calls and make sophisticated decisions based on intermediate results.

**What is Interleaved Thinking?**

Traditional AI agents execute tools sequentially without reasoning between calls. Interleaved thinking allows Claude to:

- **Reason about tool results** before deciding the next action
- **Chain multiple tool calls** with reasoning steps in between
- **Make nuanced decisions** based on intermediate findings
- **Adjust strategy dynamically** as new information becomes available

**How It Works in Error Analysis:**

```
1. Agent receives error: "KeyError: 'email'"
   ↓ [Thinking: This looks like a missing field error, I should check the logs first]
   
2. Calls fetch_cloudwatch_logs()
   ↓ [Thinking: Logs show user_data={'profile':...,'age':30} - no email field.
      Now I need the source code to see how email is accessed]
   
3. Calls fetch_source_code()
   ↓ [Thinking: Code does user_data['email'].lower() without validation.
      Let me search the knowledge base for validation patterns]
   
4. Calls search_knowledge_base()
   ↓ [Thinking: Found 3 relevant docs on input validation. I now have enough
      evidence to provide a comprehensive analysis with high confidence]
   
5. Generates analysis with root cause + specific fix + confidence score
```

**Configuration:**

```python
model = BedrockModel(
    model_id="us.anthropic.claude-sonnet-4-20250514-v1:0",
    additional_request_fields={
        # Enable interleaved thinking beta feature
        "anthropic_beta": ["interleaved-thinking-2025-05-14"],
        # Configure reasoning parameters
        "reasoning_config": {
            "type": "enabled",
            "budget_tokens": 3000  # Thinking budget for complex analysis
        }
    }
)
```

**Benefits for Error Analysis:**

- **Better tool selection** - Reasons about which tools to call based on error type
- **Efficient investigation** - Avoids unnecessary tool calls by reasoning about results
- **Higher accuracy** - Makes connections between source code, logs, and documentation
- **Contextual recommendations** - Provides fixes tailored to the specific error context

**Token Budget:**

The 3000-token thinking budget allows Claude to perform deep reasoning about complex errors. This is separate from the output tokens and enables thorough analysis without compromising response quality.

### Tool Execution Results

Each tool captures and stores its results for confidence calculation:

- **Source Code Tool**: Tries S3 source bucket first, falls back to Lambda deployment package
- **CloudWatch Logs Tool**: Uses request ID to extract exact execution logs (START to REPORT)
- **Knowledge Base Tool**: Searches with relevance scoring and confidence assessment

## Deployment Notebook Features

The `deploy-agent.ipynb` notebook provides:

### Helper Functions

**Smart Log Waiting:**
```python
wait_for_analyzer_logs(analyzer_function_name, request_id, timeout=60)
# Polls CloudWatch every 5s until analysis completes or timeout
```

**Accurate Log Retrieval:**
```python
get_analyzer_logs(analyzer_function_name, business_request_id)
# Finds analyzer execution that processed specific business request
# Extracts full execution logs using START/REPORT markers
```

**Analysis History:**
```python
# View all analysis results in formatted table
display(df[['error_id', 'timestamp', 'function_name', 
            'agent_analysis', 'analysis_duration_mm_ss']])
```

### Environment Configuration

```python
# Toggle between Claude models
USE_SONNET_4 = True  # Claude Sonnet 4 with extended thinking
# USE_SONNET_4 = False  # Claude 3.7 Sonnet

# Control DynamoDB storage
STORE_CLOUDWATCH_LOGS = True
STORE_SOURCE_CODE = True
```

## Knowledge Base

The system includes curated documentation for the AI agent:

- **common-errors.md** - Frequent Lambda error patterns and solutions
- **lambda-error-patterns.md** - AWS Lambda-specific issues
- **user-data-validation-patterns.md** - 9 test scenarios with detailed fixes
- **troubleshooting-guide.md** - Step-by-step debugging procedures
- **best-practices.md** - AWS Lambda development best practices

The Knowledge Base is indexed by Amazon Bedrock and searched during error analysis to provide context-aware recommendations.

## Advanced Features

### Dual Source Code Retrieval

The agent tries two methods to fetch source code:

1. **S3 Source Bucket** (primary) - Direct access to original source files
2. **Lambda Deployment Package** (fallback) - Downloads and extracts ZIP from Lambda function

This ensures source code is always available for analysis, even if S3 bucket is not configured.

### Request ID Filtering

CloudWatch logs are filtered using Lambda request IDs to get exact execution logs:

```python
def extract_execution_logs(all_events, request_id):
    """Extract logs between START and REPORT markers"""
    execution_events = []
    start_found = False
    
    for event in all_events:
        if f"START RequestId: {request_id}" in event['message']:
            start_found = True
        if start_found:
            execution_events.append(event)
        if f"REPORT RequestId: {request_id}" in event['message']:
            break
    
    return execution_events
```

### Historical Analysis

All analysis results are stored in DynamoDB with:
- Original error event and response
- Tool execution results (source code, logs, KB context)
- Confidence scores and evidence quality metrics
- Analysis duration and timing information
- Extracted recommendations

This enables:
- Pattern recognition across similar errors
- Analysis quality improvement over time
- Debugging and troubleshooting of the AI agent itself

## Environment Variables

**Sample Business Function:**
- `EVENT_BUS_NAME` - EventBridge bus for error events

**Error Analyzer Agent:**
- `KNOWLEDGE_BASE_ID` - Amazon Bedrock Knowledge Base ID
- `SOURCE_CODE_BUCKET` - S3 bucket with Lambda source code
- `DYNAMODB_TABLE_NAME` - Table for storing analysis results
- `USE_SONNET_4` - Toggle between Claude models (default: true)
- `STORE_CLOUDWATCH_LOGS` - Store logs in DynamoDB (default: true)
- `STORE_SOURCE_CODE` - Store source code in DynamoDB (default: true)

## Cost Considerations

**Primary cost driver:** Amazon Bedrock API calls for Claude Sonnet 4

The main cost comes from Bedrock model invocations. Actual costs depend on:
- Input tokens (error context, source code, logs, KB results)
- Output tokens (analysis response)
- Thinking tokens (3000 token budget for reasoning)
- Number of tool calls per analysis (typically 2-3)

**Other AWS service costs:**
- **CloudWatch Logs**: Ingestion and storage (existing logs, minimal incremental cost)
- **DynamoDB**: On-demand pricing for storing analysis results
- **S3**: Storage for source code and Knowledge Base documents
- **Lambda**: Execution time (typically <30 seconds per analysis)
- **EventBridge**: Event delivery (minimal)

**Cost optimization strategies:**

1. **Reduce thinking token budget** (3000 → 1500) for simpler errors
2. **Use Claude 3.7 Sonnet** instead of Sonnet 4 (lower per-token cost)
3. **Implement caching** for similar error patterns
4. **Limit Knowledge Base search results** (5 → 3 documents)
5. **Use DynamoDB provisioned capacity** for predictable workloads
6. **Set up cost alerts** in AWS Cost Explorer

**Pricing resources:**
- [Amazon Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
- [Amazon DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)

**Note:** Always check current AWS pricing for your region and usage patterns. Use AWS Cost Explorer to monitor actual costs.

## Troubleshooting

### CDK Deployment Issues

**Problem**: AWS credentials not configured
```bash
# Configure AWS CLI with your credentials
aws configure

# Or use AWS SSO
aws sso login --profile your-profile
export AWS_PROFILE=your-profile
```

**Problem**: Insufficient IAM permissions
```bash
# Ensure your AWS user/role has permissions for:
# - CloudFormation (create/update stacks)
# - Lambda (create functions, layers)
# - IAM (create roles, policies)
# - S3 (create buckets, upload objects)
# - EventBridge (create rules, event buses)
# - DynamoDB (create tables)
# - Bedrock (access to Knowledge Base and models)
```

**Problem**: Bedrock model access denied
```bash
# Enable Claude models in AWS Console:
# https://console.aws.amazon.com/bedrock/home#/modelaccess
# Required models:
# - Claude 3.7 Sonnet
# - Claude Sonnet 4
```

**Problem**: CDK bootstrap required
```bash
# Bootstrap CDK in your AWS account/region (one-time setup)
cdk bootstrap aws://ACCOUNT-ID/REGION
```

### Agent Not Analyzing Errors

**Check EventBridge rule:**
```bash
aws events list-rules --name-prefix LambdaErrorAnalysis
```

**Check Lambda logs:**
```bash
aws logs tail /aws/lambda/LambdaErrorAnalysis-error-analyzer-agent --follow
```

**Verify Knowledge Base:**
```bash
aws bedrock-agent get-knowledge-base --knowledge-base-id <KB_ID>
```

## Security Best Practices

1. **Use `expose_errors=False` in production** - Prevents sensitive error details in responses
2. **Implement least-privilege IAM roles** - Lambda functions have minimal required permissions
3. **Enable CloudWatch Logs encryption** - Encrypt logs at rest
4. **Use VPC endpoints** - Keep traffic within AWS network
5. **Rotate credentials regularly** - Use AWS Secrets Manager for sensitive data
6. **Enable AWS CloudTrail** - Audit all API calls
7. **Review Knowledge Base content** - Ensure no sensitive information in documentation

## Performance Optimization

**Reduce Analysis Time:**
- Decrease thinking token budget (3000 → 1500)
- Use Claude 3.7 Sonnet instead of Sonnet 4
- Limit Knowledge Base search results (5 → 3)
- Reduce source code size limits

**Improve Accuracy:**
- Increase thinking token budget (3000 → 5000)
- Add more documentation to Knowledge Base
- Include historical error patterns
- Fine-tune confidence scoring weights

## Future Enhancements

- [ ] Multi-turn conversation support for clarification
- [ ] Automatic fix application (with approval workflow)
- [ ] Integration with incident management systems
- [ ] Slack/Teams notifications with analysis
- [ ] Custom error pattern learning
- [ ] Multi-language support (Java, Node.js, Go)
- [ ] Real-time analysis dashboard
- [ ] A/B testing different AI models

## Contributing

Contributions welcome! Areas for improvement:

- Additional test scenarios
- More Knowledge Base documentation
- Support for other Lambda runtimes
- Integration with monitoring tools
- Performance optimizations
- Cost reduction strategies

## License

This project is licensed under the Apache License 2.0 - see the LICENSE file for details.

## Resources

- **Strands Agents SDK**: https://strandsagents.com/
- **AWS CDK Documentation**: https://docs.aws.amazon.com/cdk/
- **Amazon Bedrock**: https://aws.amazon.com/bedrock/
- **Claude Models**: https://www.anthropic.com/claude

---

*Built with [Strands Agents SDK](https://strandsagents.com/) and Amazon Bedrock*
