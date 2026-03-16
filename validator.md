// index.js

    const express = require('express');
    const userRouter = require('../routes/user.routes');

    const app = express();
    app.use(express.json());

    app.use('/api/auth', userRouter);
    app.listen(3000);



// routes/user.routes.js

    const express = require('express');
    const router = express.Router();

    const validate = require('../middleware/validate');
    const { createUserSchema } = require('../validators/user.validator');
    const userController = require('../controllers/user.controller');

    router.post( '/register',  validate(createUserSchema),  userController.createUser);

    module.exports = router;


// validators/user.validator.js

    const { checkSchema } = require('express-validator');
    exports.createUserSchema = checkSchema({
	    username: {
		    in: ['body'],
		    notEmpty: {
		      errorMessage: "Username is required"
		    },
		    isLength: {
		      options: { min: 3 },
		      errorMessage: "Username must be at least 3 characters"
		    },
		    custom: {
		        options: async (value) => {
		          	const user = await User.findOne({ username: value });
		          	if (user) {
		            	throw new Error("Username already exists");
		          	}
		          	return true;
		        }
		    }
	    },

	    email: {
		    in: ['body'],
		    isEmail: {
		      errorMessage: "Invalid email address"
		    },
		    custom: {
		        options: async (value) => {
		          	const user = await User.findOne({ email: value });
			        if (user) {
			            throw new Error("Email already exists");
			        }
			        return true;
		        }
		    }
	    },

	    password: {
        	in: ['body'],
	        isLength: {
	          options: { min: 6 },
	          errorMessage: 'Password must be at least 6 characters'
	        }
	    },

	    age: {
		    in: ['body'],
		    isInt: {
		      options: { min: 18 },
		      errorMessage: "Age must be at least 18"
		    }
	    },

      	'addresses.*.street': {
      		in: ['body'],
		    notEmpty: true,
	    },

	    'addresses.*.number': {
		    in: ['body'],
		    isInt: true,
	    },
    });



// middleware/validate.js

    const { validationResult } = require('express-validator');
    const validate = (validations) => {
	    return async (req, res, next) => {

		    for (const validation of validations) {
		      await validation.run(req);
		    }

		    const errors = validationResult(req);
		    if (!errors.isEmpty()) {
			    return res.status(400).json({
				    success: false,
				    errors: errors.array()
			    });
		    }

		    next();
	    };
    };

    module.exports = validate;


// controllers/user.controller.js

    exports.createUser = async (req, res) => {
      const { name, email, password } = req.body;

      res.json({
        message: "User created",
        data: { name, email }
      });
    };
