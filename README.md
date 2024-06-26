# MongoDB Aggregation Pipelines  

Dummy data that u will need : [click here](https://gist.github.com/hiteshchoudhary/a80d86b50a5d9c591198a23d79e1e467)  

## Topics:Group,Match,Project and Lookup  

1.How many users are active ?  

    [  
      {  
        $match: {  
          isActive:true  
        }  
      },  
      {  
        $count: 'activeUsers'  
      }  
    ]  

2.What is the average age of all the users ?  

    [
        {
            $group: {
                _id:null, //everyone is inside this doc now
                averageAge:{
                    $avg:"$age"
                }
            }
        }
    ]

3.List the top 2 most common fruits among users  

    [
        {
            $group: {
                _id: "$favoriteFruit",
                count:{
                    $sum:1,
                }
            }
        },
        {
            $sort: {
                count: -1
            }
        },
        {
            $limit: 2
        }
    ]

4.Find the total number of males and females.  

    [
        {
            $group: {
                _id: "$gender",
                genderCount:{
                    $sum:1
                }
            }
        }
    ]

5.Which country has the highest number of registered users?  

    [
        {
            $group: {
                _id: "$company.location.country",
                userCount:{
                    $sum:1
                }
            }
        },
        {
            $sort: {
                userCount: -1
            }
        },
        {
            $limit: 1
        }
    ]

6.List all the unique eye colors present in the collection.  

    [
        {
            $group: {
                _id: "$eyeColor",
            }
        }
    ]

and if u want count  

    [
        {
            $group: {
                _id: "$eyeColor",
                userCount:{
                    $sum:1
                }
            }
        }
    ]

7.What is average number of tags per user?  

Dealing with array in aggregation { some data, [tags] }

    [
        {
            $unwind: {
                path: "$tags",
            }
        },
        {
            $group: {
                _id: "$_id",
                numberOfTags:{
                    $sum: 1
                }
            }
        },
        {
            $group: {
                _id: null,
                averageNumberOfTags:{
                $avg: "$numberOfTags"
                }
            }
        }
    ]

alternative  

    [
        {
            $addFields: {
                numberOfTags: {
                    $size: {
                        $ifNull:["$tags",[]]
                    }
                }
            }
        },
        {
            $group: {
                _id: null,
                averageNumberOfTags:{
                    $avg: "$numberOfTags"
                }
            }
        }
    ]

8.How many users have 'enim' as one of the tags?  

    [
        {
            $match: {
                tags:"enim"
            }
        },
        {
            $count: 'userWithEnimTag'
        }
    ]

9.What are the names and age of users who are inactive and have 'velit' as a tag ?  

    [
        {
            $match: {
                isActive:false,tags:"velit"
            }
        },
        {
            $project: {
                name:1,
                age:1
            }
        }
    ]

10.How many users have phone number starting with '+1(940)'?  

    [
        {
            $match: {
                "company.phone": /^\+1 \(940\)/
            }
        },
        {
            $count: 'usersWithSpecialPhoneNumber'
        }
    ]

11.Who has registered most recently ?  

    [
        {
            $sort: {
                registered: -1
            }
        },
        {
            $limit: 1
        },
        {
            $project: {
                name:1,
                registered:1,
                favoriteFruit:1,
            }
        }
    ]

12.Categorize users by their favorite fruit.  

    [
        {
            $group: {
                _id: "$favoriteFruit",
                users:{
                    $push:"$name"
                }
            }
        }
    ]

13.How many users have 'ad' as the second tag in their list of tags?  

    [
        {
            $match: {
                "tags.1":"ad"
            }
        },
        {
            $count: 'secondTagAd'
        }
    ]

14.Find users who have both 'enim' and 'id' as their tags.  

    [
        {
            $match: {
                tags:{
                    $all:['enim','id']
                }
            }
        }
    ]

15.List all the companies located in the USA with their corresponding user count.  

    [
        {
            $match: {
                "company.location.country":"USA"
            }
        },
        {
            $group: {
                _id: "$company.title",
                userCount:{
                    $sum:1,
                }
            }
        }
    ]

16.Take data of authors when u r on books collection  

lookup-joining
addFields makes it more easy to get the data

    [
        {
            $lookup: {
                from: "authors",
                localField: "author_id",
                foreignField: "_id",
                as: "author_details"
            }
        },
        {
            $addFields: {
                author_details:{
                    // $first:"$author_details",
                    $arrayElemAt:["$author_details",0] //more used
                }
            }
        }
    ]
