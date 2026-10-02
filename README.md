# API Testing Practice

## Base URL
https://jsonplaceholder.typicode.com

## Endpoints to Test

### Posts
GET /posts
GET /posts/1
POST /posts
PUT /posts/1
PATCH /posts/1
DELETE /posts/1

### Users
GET /users
GET /users/1

### Comments
GET /comments?postId=1

## Sample POST Request

POST /posts

Request Body:
{
  "title": "API Testing",
  "body": "Testing API using Codex",
  "userId": 1
}

## Testing Scope

Test the APIs for:
- Positive scenarios
- Negative scenarios
- Valid and invalid IDs
- Missing required fields
- Empty request body
- Invalid data types
- Status codes
- Response body
- Response headers
- Error handling

## Expected Output

Create a test report containing:
- Test Scenario
- Endpoint
- HTTP Method
- Test Data
- Expected Result
- Actual Result
- Pass/Fail
- Defect/Observation
