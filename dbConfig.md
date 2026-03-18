//backend/server.js

	require('dotenv').config();
	const express = require('express');
	const cors = require('cors');
	const dbConnect = require('./config/dbConnect');

	const app = express();
	const PORT = process.env.PORT || 8000;	
    app.use(express.json());
	app.use(express.urlencoded({extended:true}));
	app.use(cors());

    app.listen(PORT,()=>{
		dbConnect();
		console.log("server is runing at PORT=>", PORT);
	})

// config/dbConnect

    const mongoose = require("mongoose");
    async function dbConnect() {
        try {
            const connect = await mongoose.connect(process.env.MONGODB_URL);
            console.log(`Database connected Successfully: ${connect.connection.host},${connect.connection.name}`);
        } catch (error) {
            console.log('Database error', error);
            process.exit(1)
        }    
    }

    module.exports = dbConnect;