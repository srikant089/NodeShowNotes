// install package

  npm i express
  npm i cors
  npm i express-rate-limit
  npm i express-slow-down
  npm i helmet

// index.js

  const express = require('express');
  const cors = require('cors')
  const rateLimit  = require('express-rate-limit');
  const slowDown  = require('express-slow-down');
  const helmet = require('helmet');

  const app = express();

  const corsOptions = {
    origin: 'http://example.com',
    credentials: true,
    optionsSuccessStatus: 200 // some legacy browsers (IE11, various SmartTVs) choke on 204
  }

  // Adds headers: Access-Control-Allow-Origin: *
  app.use(cors(corsOptions))



  //site block for Ip some time (express-rate-limit)
    const limiter = rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      limit: 10, // Limit each IP to 100 requests per `window` (here, per 15 minutes).
      message:"Too many request from his IP, Please try again  later."
    })

    // Apply the rate limiting middleware to all requests.
    app.use(limiter)


  //site go slow dwon (express-slow-down')
    const apiLimiter = slowDown({
      windowMs: 15 * 60 * 1000, // 15 minutes
      delayAfter: 1, // Allow only one request to go at full-speed.
      delayMs: (hits) => hits * hits * 1000, // 2nd request has a 4 second delay, 3rd is 9 seconds, 4th is 16, etc.
    })

    // Apply the delay middleware to all requests.
      app.use(apiLimiter)

    // Apply the delay middleware to API calls only.
      app.use('/api', apiLimiter)


  // helmet: Disable the Content-Security-Policy and X-Download-Options headers
    app.use(
      helmet({
        contentSecurityPolicy: false,
        xDownloadOptions: false,
      }),
    );

  app.listen(3000);
