//To create database
use zen_class

//to create collections
db.createCollection("user")
db.createCollection("code_kata")
db.createCollection("attendance")
db.createCollection("topics")
db.createCollection("tasks")
db.createCollection("company_drives")
db.createCollection("mentors")

//to insert document in user collection

db.user.insertMany([{"name":"raj","email":"raj@gmail.com","gender":"M"},{"name":"Rose","email":"rose@gmail.com","gender":"F"},{"name":"Rajesh","email":"rajesh@gmail.com","gender":"M"}])

//to insert data into code kata collection		

db.code_kata.insertMany([{"userId":ObjectId('68826901b48dfa5c8e13d306'),"problemSolved":100},{"userId":ObjectId('68826901b48dfa5c8e13d307'),"problemSolved":200},{"userId":ObjectId('68826901b48dfa5c8e13d308'),"problemSolved":50}])

//to insert data into attendance

db.attendance.insertMany([{"userId":ObjectId('68826901b48dfa5c8e13d306'),"date":ISODate("2020-10-20"),"present":true},{"userId":ObjectId('68826901b48dfa5c8e13d307'),
"date":ISODate("2020-10-02"),"present":false},{"userId":ObjectId('68826901b48dfa5c8e13d308'),"date":ISODate("2020-10-28"),"present":true}])

//to insert data into topics

db.topics.insertMany([{"topic": "JavaScript Basics","date": ISODate("2020-10-10")},{"topic": "JavaScript array","date": ISODate("2020-10-19")}])

//to insert data into tasks

db.tasks.insertMany([{"task_name":"loop","submitted":true,"userId":ObjectId('68826901b48dfa5c8e13d308'),"date":ISODate("2020-10-11")},{"task_name":"DOM","submitted":false,"userId":ObjectId('68826901b48dfa5c8e13d306'),"date":ISODate("2020-10-31")}])

//to insert data into company_drives

db.company_drives.insertMany([
  {
    "company": "Google",
    "date": ISODate("2020-10-18"),
    "students": [
      ObjectId("68826901b48dfa5c8e13d306"),
      ObjectId("68826901b48dfa5c8e13d307")
    ]
  },
  {
    "company": "Amazon",
    "date": ISODate("2020-10-25"),
    "students": [
      ObjectId("68826901b48dfa5c8e13d307"),
      ObjectId("68826901b48dfa5c8e13d308")
    ]
  },
  {
    "company": "Facebook",
    "date": ISODate("2020-11-05"),
    "students": [
      ObjectId("68826901b48dfa5c8e13d307")
    ]
  }
]);

db.mentors.insertMany([
  {
    _id: ObjectId("652000000000000000000001"),
    name: "Mentor A",
    mentees: [
      ObjectId("651000000000000000000001"),
      ObjectId("651000000000000000000002"),
      ObjectId("651000000000000000000003"),
      ObjectId("651000000000000000000004"),
      ObjectId("651000000000000000000005")
    ]
  },
  {
    _id: ObjectId("652000000000000000000002"),
    name: "Mentor B",
    mentees: [
      ObjectId("651000000000000000000006"),
      ObjectId("651000000000000000000007"),
      ObjectId("651000000000000000000008"),
      ObjectId("651000000000000000000009"),
      ObjectId("651000000000000000000010"),
      ObjectId("651000000000000000000011"),
      ObjectId("651000000000000000000012"),
      ObjectId("651000000000000000000013"),
      ObjectId("651000000000000000000014"),
      ObjectId("651000000000000000000015"),
      ObjectId("651000000000000000000016"),
      ObjectId("651000000000000000000017"),
      ObjectId("651000000000000000000018"),
      ObjectId("651000000000000000000019"),
      ObjectId("651000000000000000000020"),
      ObjectId("651000000000000000000021") 
    ]
  },
  {
    _id: ObjectId("652000000000000000000003"),
    name: "Mentor C",
    mentees: [] 
  }
]);


1)Find all the topics and tasks which are thought in the month of October

const start = new Date("2020-10-01")
const end = new Date("2020-10-31")
db.topics.find({date:{$gte: start,$lte:end}})

1.1)Find all the tasks which are thought in the month of October
const start = new Date("2020-10-01")
const end = new Date("2020-10-31")
db.tasks.find({date:{$gte:start,$lte:end}})


2)Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020
const from_date = new Date("2020-10-15")
const to_date =  new Date("2020-10-31")
db.company_drives.find({date:{$gt:from_date,$lt:to_date}})

3)Find all the company drives and students who are appeared for the placement.

db.company_drives.aggregate([{$lookup:{from:"user",localField:"students",foreignField:"_id",as:"attended_student"}},{$project:{company:1,date:1,attended_student:{name:1}}}])


4)Find the number of problems solved by each user in codekata

db.code_kata.aggregate([{
			$lookup:{from:"user",localField:"userId",foreignField:"_id",as:"codekata_user"}},{$unwind: "$codekata_user"},{$project:{problemSolved:1,name:"$codekata_user.name"}}])


5)Find all the mentors with who has the mentee's count more than 15



db.mentors.find({
	$expr: { $gt: [{$size: "$mentees"}, 15]}
})


6)Find the number of users who are absent and task is not submitted  between 15 oct-2020 and 31-oct-2020

db.attendance.aggregate([
  {
    $match: {
      date: { $gte: ISODate("2020-10-15"), $lte: ISODate("2020-10-31") },
      present: false
    }
  },
  {
    $lookup: {
      from: "tasks",
      let: { user_id: "$userId", att_date: "$date" },
      pipeline: [
        {
          $match: {
            $expr: {
              $and: [
                { $eq: ["$userId", "$$user_id"] },
                { $eq: ["$submitted", false] },
                { $gte: ["$date", ISODate("2020-10-15")] },
                { $lte: ["$date", ISODate("2020-10-31")] }
              ]
            }
          }
        }
      ],
      as: "unsubmittedTasks"
    }
  },
  {
    $match: {
      unsubmittedTasks: { $ne: [] }
    }
  },
  {
    $group: {
      _id: "$userId"
    }
  },
  {
    $count: "absentAndNotSubmittedUsers"
  }
]);
