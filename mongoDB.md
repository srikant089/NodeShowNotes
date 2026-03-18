// data types support

    ObjectId
    String
    Integer
    Double
    Boolean
    Array
    Object/Embedded Docu
    Date
    Timestamp
    null

// find Query

    db.cars.find();
    db.cars.findOne();
    db.cars.find({},{model:1});
    db.cars.find({},{model:1, _id:0});
    db.cars.find({},{fuel_type:"Petrol"});
    db.cars.find({},{"engine.type":"Turbochared"});

// inser Query

    db.cars.insertOne({date:new Date()});
    db.cars.insertOne({TS:new Timestamp()});

// update Query 

    db.cars.updateOne(
        { model: "Nexon" }, // Filter to find  all document where
        { $set: { fuel_type: "Electtic" } } //update the fuel_type
    );
    
    db.cars.updateMany(
        { fuel_type: "Diesel" }, // Filter to find  all document where
        { $set: { alloys: "yes" } } //update the alloys
    );
    
    db.cars.updateone(
        { model: "Nexon" }, // Filter to find  all document where
        { $set: { 
            "engine.cc": 1300,
            "engine.torque": "180 Nm",
        } } //update the fuel_type
    );

    // updating Array
        db.cars.updateOne(
            { model: "Nexon" },
            { $push: { features: "Wireless Charging" } }
        );

        // already key exists
        db.cars.updateOne(
            { model: "Nexon" },
            { $pull: { features: "Wireless Charging" } }
        );

    // update Multiple values Array
        // already key exists
        db.cars.updateOne(
            { model: "Nexon" },
            { $push: { features: { $each: ["Wireless Charging", "Voice Control" ] } } }
        );

    // updateMany
        db.cars.updateMany(
            {},
            { $set: { color: "red" } }
        );

        db.cars.updateMany(
            { model: "Nexon" },
            { $set: { color: "red" } }
        );

        // upset is combination of the operations "update" and "insert"
        db.cars.updateMany(
            { model: "Venue" },
            { $set: { Maker: "Hyundai" } },
            { upset: true }
        );

// remove filed

    db.cars.updateOne(
        { model: "Nexon" },
        { $unset: { color: "" } }
    );

// Delete Query

    db.cars.deleteOne(
        { fuel_type: "Diesel" }
    );

    db.cars.deleteMany(
        { fuel_type: "Diesel" }
    );


// Operators

    //relational Operators ($eq: =, $lt: <, $gt: >, $lte: <=, $gte: >=, $ne: !=)
        db.cars.find({ "engine.cc": 1493 });
        db.cars.find({ "engine.cc":{ $gt:1400 } });

    // logical Operators ($in: [],  $nin: ![])
        db.cars.find({ "engine.cc":{ $in:[1498, 2179] } });
        db.cars.find({ "engine.cc":{ $nin:[1498, 2179] } });

    // logical Operators ($and: &&,  $or: ||, $nor, $not )
        db.cars.find({ 
            $and: [
                { "fuel_type": "Diesel" },
                { "engine.type": "Turbocharged" },
                { "sunroof": true }
            ] 
        });

        db.cars.find({ 
            $or: [
                { "fuel_type": "Diesel" },
                { "engine.type": "Turbocharged" }
            ] 
        });

        db.cars.find({ 
            $nor: [
                { "fuel_type": "Diesel" },
                { "engine.type": "Turbocharged" }
            ] 
        });

// Element Operators ($exists, $type)

    db.cars.find({ "fuel_type": {$exists: true} });

// Array Operators ($size, $all)

    db.cars.find({ "feature": {$size: 4} });
    db.cars.find({ "feature": {$all: ['Bluetooth','Sunroof']} });


// Cursor Methods (count(), sort(), limit(), skip())

    db.cars.find().count();
    db.cars.find({ "fuel_type": "Diesel" }).count();

    db.cars.find({},{ _id:0, model:1});
    db.cars.find({},{ _id:0, model:1}).sort({model:1});

    db.cars.find().limit(2);

    db.cars.find().skip(2);


// Aggregation Framewrok (filtering/match, grouping, sorting, reshaping, and summarizing via pipeline)

   // ($match, $group, $project, $sort, $limit, $unwind, $lookup, $addFields, $count, $skip)

    Example: db.collection.aggregate([
        {
            $group: {
                _id:"$maker", //field name
                totalAmount: { $sum: "$price" },
                averageAmount: { $avg: "$price" },
                minAmount: { $avg: "$price" },
                maxAmount: { $max: "$price" },
            }
        }
    ]);
    
    db.cars.aggregate([
        {
            $group: {
                _id:"$maker", //field name
                TotalCars: { $sum:1 }, // { $sum: "$ca"}
                AvgPrice: { $avg: "$price"}, // { $avg: "$price"}
            }
        }
    ]);

    db.cars.aggregate([
        {
            $group: {
                _id:"$fuel_type", //field name
                TotalCars: { $sum:1 }, // find the number
            }
        }
    ]);



    db.cars.aggregate([
        {
            $group: {
                _id:"$maker", //field name
                TotalCars: { $sum:1 }, // { $sum: "$ca"}
                AvgPrice: { $avg: "$price"}, // { $avg: "$price"}
            }
        }
    ]);

    // Match(as filter)

    db.cars.aggregate([
        {
            $match: {
                "maker": "Hyundai",
                "engine.cc": {$gt: 1200 },
            }
        }
    ]);

    // Count

    db.cars.aggregate([
        {
            $match: {
                "maker": "Hyundai",
            }
        },
        {
            $count: "Total_cars"
        }
    ]);

    // Count no. of diesel and petrol cars of Hyundai brand

    db.cars.aggregate([
        {
            $match: {
                "maker": "Hyundai",
            }
        },
        {
            $group: {
                _id:"$fuel_type", //field name
                TotalCars: { $sum:1 }, // find the number
            }
        }
    ]);


    // Project

    // Find all the Hyundai cars and only show Maker, Model and Fuel_type
        db.cars.aggregate([
            {
                $match: {
                    "maker": "Hyundai",
                }
            },
            {
                $project: {
                    maker: 1,
                    model: 1,
                    fuel_type: 1,
                    _id:0
                }
            },
            {
                $sort: {
                    maker: 1 //asc:1, desc:-1
                }
            },
        ]);


    // sortByCount

    db.cars.aggregate([
        {
            $sortByCount: "$maker"
        },
    ]);

    //unwind

    db.cars.aggregate([
        {
            $unwind: "$owners"
        },
    ]);

    // string operators ($concat, $toUpper, $toLower, $regexMatch, $ltrim, $split)
        //toUpper and concat
        db.car.aggregate([
            {
                $match: {
                    "maker": "Hyundai",
                }
            },
            {
                $project: {
                    _id:0,
                    carName: { $toUpper: { $concat: ["$maker", " ", "$model"] } },
                }
            }
        ]);

    // regexMach
    db.car.aggregate([
        {
            $project: {
                _id:0,
                model:1,
                is_diesel: { 
                    $regexMach: { 
                        input: "$fuel_type",
                        regex: "Die"
                    } 
                },
            }
        }
    ]);

    // out after aggregating store the result in an other collection "hyundai_cars"

    db.car.aggregate([
        {
            $match: {
                "maker": "Hyundai",
            }
        },
        {
            $project: {
                _id:0,
                carName: { $toUpper: { $concat: ["$maker", " ", "$model"] } },
            }
        },
        {
            $out: "hyundai_cars"
        }
    ]);


    // airthmetic Operations($add, $subract, $divide, $multiply, $round, $abs, $ceil)

    db.car.aggregate([
        {
            $project: {
                _id:0,
                model:1,
                new_price: { $add: ["$price", "50000"] },
            }
        }
    ]);

    // addFields Stage and round & divide the price

    db.car.aggregate([ 
        {
            $project: {
                _id:0,
                model:1,
                price: 1,
            }
        },
        {
            $addFields: {
                price_in_lac: {
                    $round: [{$divide:["$price", 100000]}, 1]
                }
            }
        }
    ]);

    // Set stage

    db.car.aggregate([
        {
            $match: {
                "maker": "Hyundai"
            }
        },
        {
            $set: {
                total_service_cost: { $sum :"$service_history.cost" }
            }
        },
        {
            $project: {
                _id:0,
                model:1,
                total_service_cost: 1,
            }
        }
    ]);


    // conditional operator ($cond,$switch)
    
    db.car.aggregate([
        {
            $match: {
                "maker": "Hyundai"
            }
        },        
        {
            $project: {
                _id:0,
                maker:1,
                model: 1,
                fuel_cate: {
                    $cond: {
                        if: { $eq: ["$fuel_type", "Petrol"]},
                            than:"Petrol Car",
                        else: "Non-Petrol Car"
                    }
                }
            }
        }
    ]);

    db.car.aggregate([ 
        {
            $project: {
                _id:0,
                maker:1,
                model: 1,
                price_cate: {
                    $switch: {
                        brances: [
                            { case: { $lt: ["$price", 500000]},
                                then: "Budget"
                            },
                            { case: { $and: [
                                        { $gte: ["$price", 500000]}, { $lt: ["$price", 1000000]} 
                                    ]},
                                then: "Midrange"
                            },
                            { case: { $gte: ["$price", 1000000]}, 
                                then: "Premium"
                            },
                       ],
                       default: "Unknown"
                    }
                }
            }
        }
    ]);