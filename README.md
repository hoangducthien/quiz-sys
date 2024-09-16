# Real-Time Vocabulary Quiz Coding Challenge
## Functional requirements
  1. User Participation:  
    * Users should be able to join a quiz session using a unique quiz ID.  
    * The system should support multiple users joining the same quiz session simultaneously.  
  2. Real-Time Score Updates:  
    * As users submit answers, their scores should be updated in real-time.  
    * The scoring system must be accurate and consistent.  
  3. Real-Time Leaderboard:  
    * A leaderboard should display the current standings of all participants.  
    * The leaderboard should update promptly as scores change.  

## Load Estimation, Data estimation, non-functional requirements    

  10M daily active user  

  100K concurrent user
  
  10 new quizzes per day, each quiz can have maximum 100 question  

  10 sessions per day, each sessions can have maximum 10K participant

  All session and participant's ranking store in cache => 10 byte each reord (userId-rank), 100K record take 1MB
  
  All questions of active session will be cached
  
  100K submitted quiz per day (with 100 answer each quiz) => 100M answers per day (approximately each record size is 100 bytes)  
  
  10GB generated and stored per day  


## DataBaseModel

<img width="991" alt="Screenshot 2024-09-18 at 16 09 15" src="https://github.com/user-attachments/assets/a0261013-ceec-4535-8775-0757ff43c862">


## REST API (authorized, user_id should be able to retrive from token)

**getActiveSession()** 
```
return sessions that not finished yet (maybe started or not)
[
  {
    sessionID: String
    startTime: TimeStamp // time that session will start
    duration: Int // session will be ended after duration (seconds)
  }
]
```

**joinSession(sessionID)**
_Return error after session finised_
_BE also register user to receive realtime leaderboard & ranking_
```
{
  sessionID: String
  startTime: TimeStamp // time that session will start
  duration: Int // session will be ended after duration (seconds)
  currentProgress: Int // current user progress
  currentScore: Int // current user score
  currentRanking: Int // current user ranking
}
```
**leaveSesson(sessionID)**
_BE also unregister user to receive realtime leaderboard & ranking_
```
{}
```
**getQuestions(sessionID, pageNo, pageSize)**
```
{
  [
    questionId: Int
    content: String
    answers: [
        {
          answerId: Int
          content: String
        }
    ]
  ]
}
```
**submitAnswer(sessionId, questionId, answerId)**
_Return error after session finised_
```
{
  questionId: String
  answerId: String 
  isCorrect: Boolean // is the answer correct
  scroe: Int // score for this answer
  currentScore: Int // current user score
}
```
getLeaderBoard(sessionId)
```
{
  sessionId: String
  currentScore: Int // current user score
  currentRanking: Int // current user ranking
  top10: [
    {
      userId: String
      userName: String
      score: String 
      rank: Int
    }
  ]
}
```

## Architecture

![quiz-ds drawio](https://github.com/user-attachments/assets/2f0c5a21-a7ac-4a79-8711-c07a978fd972)

## Component Description

SQL Database: to store quiz, question, answer, session, user submited answer

Cache: save user ranking data, question data for active session 

Kafka: message queue for leader board update event

Quiz service: serve REST APIs

Realtime Service: send leaderboard realtime data to connected client

Load balancer: load balace for Quiz service

Connection Service: Load balance for Realtime service

Client: mobile app 

## Data Flow

![flow-final](https://github.com/user-attachments/assets/88e84222-c2cb-453f-a8b0-c073905a90b5)


## Technologies and Tools
1. BE:
   
    Database: SQL/NoSQL => I choose SQL for better for data consistency, less complex in development time. (NoSQL is easier for scaling but need trade off between performance & storing redundant data, more complex in development)
  
    Cache: redis/memchace => I choose redis because more popular, more powerfull, more support
  
    MessageQueue: Kafka/RabbitMQ => I choose Kafka because more popular, more familiar
  
    Realtime: MQTT/Websocket => I choose websocket because lighter, simple/fast implementation
  
    Load balancer: nginx
  
3. Client:
   
   Native/cross platform/Web-based => I choose native, better performance & user experience  



