//backend/server.js

	require('dotenv').config();
	const express = require('express');
	const cors = require('cors');
	const cookieParser = require('cookie-parser');
	const dbConnect = require('./config/dbConnect');

	const app = express();
	const PORT = process.env.PORT || 8000;	

	// MIDDLEWARE IMPORT

	app.use(express.json());
	app.use(express.urlencoded({extended:true}));
	app.use(cors());
	app.use(cookieParser());

	const corsOptions = {
		origin: "http://localhost:5173",
		credentials: true,
		exposedHeaders:"Authorization"
	}

	app.use(cors(corsOptions));	

	app.get("/test", (req, res) => {
		res.sed({message:"welcome, its workring"})
	});


	// Auth Middleware role based
    const authMiddleware = require('./middlewares/authMiddleware');

    // ROUTER IMPORT
	const authRouter = require('./routes/auth.router');
	const userRouter = require('./routes/user.router');
	const adminRouter = require('./routes/admin.router');

	// ROUTERS
	app.use("/api/auth", authRouter);
	app.use("/api/user", authMiddleware(['USER']), userRouter);
	app.use("/api/admin", authMiddleware(['ADMIN']), adminRouter);


	app.listen(PORT,()=>{
		dbConnect();
		console.log("server is runing at PORT=>", PORT);
	})

//backend/config/dbConnection.js

	const { default: mongoose } = require("mongoose")
	const dbConnect = async() => {
		try {
			const connect = await mongoose.connect(process.env.MONGO_DB);
			console.log(`Database connected Successfully: ${connect.connection.host},${connect.connection.name}`);
		} catch (error) {
			console.log('Database error', error);
			process.exit(1)
		}    
	}

	module.exports = dbConnect;


//backend/models/user.model.js

	const mongoose = require('mongoose');
	const userSchema = new mongoose.Schema({
		email: { type:String, required:true},
		password: { type:String, required:true},
		role: { type:String, required:true, default: 'USER'},
		createdAt: { type:Date, default: new Date()}
	})

	module.exports = mongoose.model("User", userSchema);


//backend/routes/auth.router.js

	const express = require('express');
	const router = express.Router();
	const {
	    register,
	    login
	} = require('../controllers/auth.controller');

	router.post('/register',register);
	router.post('/login',login);
	module.exports = router;
	

//backend/routes/user.router.js

	const express = require('express');

	const router = express.Router();
	const {
	    adminProfile,
	    profile

	} = require('../controllers/user.controller');

	router.get('/', profile);

	module.exports = router;


//backend/routes/admin.router.js

	const express = require('express');

	const router = express.Router();

	const {
	    adminProfile,
	    users
	} = require('../controllers/admin/user.controller');

	router.get('/users', users);

	router.get('/', adminProfile);

	module.exports = router;



//backend/middlewares/auth.middleware.js

	require('dotenv').config();
	const jwt = require('jsonwebtoken');

	// Role:[USER, ADMIN]
	const authMiddleware = (roles=[]) => { return (req, res, next) => {
		try {
			
			const token = req.header("Authorization")?.replace("Bearer ", "");
			if (!token) {
				res.status(401).json({success:false, message:"Invalid token"});
			}

			const decoded = jwt.verify(token, process.env.JWT_SECRET);
			if (decoded) {
				req.user = decoded;

				if(roles.length > 0 && !roles.includes(req.user.role)) {
					return res.status(403).json({success:false, message:"Access Denied."})
				}
			}
			next();
		} catch (error) {
	        console.log("ERROR in authMiddleware", error);
	        res.status(500).json({success:false, message:"Internal Server Error."});
	    }	

	}}

	module.exports = authMiddleware;


//backend/controllers/auth.controller.js

	require('dotenv').config();
	const bcrypt = require('bcrypt');
	const jwt = require('jsonwebtoken');
	const { Validator } = require('node-input-validator');
	const User = require('../models/user.model');
	
	const register = async(req, res) => {
		try {
			
			// if validation fails, this will auto abort request with status code 422 and errors in body
			const v = new Validator(req.body, {
				username: 'required|maxLength:15',
				email: 'required|email',
				password: 'required'
			});

			// in case validation fails
			const matched = await v.check();
			if (!matched) {
				return res.status(422).send({ Status: false, error: v.errors });
			}

			const { username, email, password, confirmPassword } = req.body;
			const user = await User.findOne({email: email});
			if(user) {
				return res.status(409).json({success:false, message:"Email is already registerd."});
			}

			if (password !== confirmPassword) {
				return res.status(400).send({ Status: false, error: "Passwords don't match." })
			}	                

			const salt = await bcrypt.genSaltSync(10);
			const hashPassword = await bcrypt.hashSync(password, salt);	                
			const newUser = new User({
				password: hashPassword,
				email: email,
				username: username,
				role: 'USER',	                    
			})

			const newUser = await newUser.save();
			console.log("Register user data: ", newUser);
			res.status(201).json({success:true, data:newUser, message:"Registered SuccessFully."});
			
		} catch (error) {
			console.log("ERROR in Register", error);
			res.status(500).json({success:false, message:"Internal Server Error."});
		}
	},

	const login = async(req, res) => {
		try {
			// if validation fails, this will auto abort request with status code 422 and errors in body
			const v = new Validator(req.body, {
				email: 'required|email',
				password: 'required'
			});

			// in case validation fails
			const matched = await v.check();
			if (!matched) {
				return res.status(422).send({ Status: false, error: v.errors });
			}

			const { email, password } = req.body;

			const user = await User.findOne({email: email});
			if(user) {
				const isAuth = bcrypt.compareSync(password, user.password);
				if (isAuth) {
					const token = await jwt.sign(
						{
							id: user._id,
							role: user.role,
						}, 
						process.env.JWT_SECRET
					);

					res.header("Authorization", token);
					res.status(200).json({
						status:true,
						message:"SuccessFully login.",
						data:{
							id: user._id,
							email: email,
							role: user.role,
						}
					})
				} else {
					res.status(422).json({success:false, message:"Email/Password invalid."});
				}
			} else {
				res.status(422).json({success:false,  message:"Email not registerd."});                
			}
		} catch (error) {
			console.log("ERROR in login", error);
			res.status(500).json({success:false, message:"Internal Server Error."});
		}
	}

	module.exports = {
		register,
		login
	}


//backend/controllers/user.controller.js

	const User = require('../models/user.model');	   

	const profile = async(req, res) => {
		try {
			
			const user = await User.findById(req.user.id).select(['-password','-createdAt']);
			if(user){
				res.status(200).json({success:true, message:'SuccessFully fetching user data.', data: user})
			}
		} catch (error) {
			console.log("ERROR in profile", error);
			res.status(500).json({success:false, message:"Internal Server Error."});
		}
	}
	
	module.exports = {	
		profile 
	}

//backend/controllers/admin/user.controller.js

	const User = require('../models/user.model');    

	const users = async(req, res) => {
		try {
			
			const page = parseInt(req.query.page) || 1;
			const limit = parseInt(req.query.limit) || 10;

			const skip = (page - 1) * limit;

			const users = await User.find()
			.skip(skip)
			.limit(limit);

			const total = await User.countDocuments();

			res.json({
				page,
				limit,
				totalPages: Math.ceil(total / limit),
				totalResults: total,
				data: users
			});
		} catch (error) {
			console.log("ERROR in profile", error);
			res.status(500).json({success:false, message:"Internal Server Error."});
		}
	}

	const adminProfile = async(req, res) => {
		try {
			
			const user = await User.findById(req.user.id).select(['-password','-createdAt']);
			if(user){
				res.status(200).json({success:true, message:'SuccessFully fetching user data.', data: user})
			}
		} catch (error) {
			console.log("ERROR in adminProfile", error);
			res.status(500).json({success:false, message:"Internal Server Error."});
		}
	}

	module.exports = {
		adminProfile,
		users	    
	}

