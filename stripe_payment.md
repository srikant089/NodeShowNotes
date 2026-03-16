// controllers/stripePayment.js


  // Create Customer (User Account)

    const stripe = require("../config/stripe");
    exports.createPaymentIntent = async (req, res) => {
      try {
        const customer = await stripe.customers.create({
          email: req.body.email,
          name: req.body.name
        });

        res.json(customer);
      } catch (err) {
        res.status(500).send(err.message);
      }
    }


  // Create PaymentIntent

    exports.createPaymentIntent = async (req, res) => {
      const { amount, currency } = req.body;
      try {

        const paymentIntent = await stripe.paymentIntents.create({
          amount: amount * 100,
          currency: currency || "usd",
          payment_method_types: ["card"]
        });

        res.json({
          clientSecret: paymentIntent.client_secret
        });

      } catch (err) {
        res.status(500).send(err.message);
      }

    };


  // Create PaymentIntent Hold Payment (Authorize Only)

    exports.createPaymentIntent = async (req,res)=>{
      const { amount } = req.body;
      try{
        const paymentIntent = await stripe.paymentIntents.create({
          amount: amount * 100, // $50 example
          currency: "usd",
          capture_method: "manual",
          payment_method_types: ["card"]

          // application_fee_amount: 10*100, // $10 platform fee

          // transfer_data: {
          //   destination: "acct_AGENCY_STRIPE_ID" // optional
          // }

        });
      }catch(err){
        res.status(500).send(err.message);
      }
    }


  // Confirm Payment

    exports.confirmPayment = async (req,res)=>{
      const { paymentIntentId } = req.body;
      try{
        const paymentIntent = await stripe.paymentIntents.confirm(
          paymentIntentId,
          {
            payment_method: paymentMethodId
          }
        );
      }catch(err){
        res.status(500).send(err.message);
      }
    }

  // Capture Payment

    exports.capturePayment = async (req,res)=>{
      const { paymentIntentId } = req.body;
      try{
        const paymentIntent = await stripe.paymentIntents.capture(paymentIntentId);
        res.json(paymentIntent);
      }catch(err){
        res.status(500).send(err.message);
      }
    }



  // Create Supplier (Stripe Connect Account)

    exports.accountCreate = async (req,res)=>{
      try {
        const account = await stripe.accounts.create({
          type: "express",
          country: "US",
          email: req.body.email,
          capabilities: {
            card_payments: { requested: true },
            transfers: { requested: true }
          }
        });

        const accountLink = await stripe.accountLinks.create({
          account: account.id,
          refresh_url: "https://yourapp.com/reauth",
          return_url: "https://yourapp.com/success",
          type: "account_onboarding",
        });

        res.json({
          accountId: account.id,
          onboardingUrl: accountLink.url
        });

      } catch (err) {
        res.status(500).send(err.message);
      }
    }

  // transfersMoney

    exports.paymentTransfers = async (req,res)=>{
      await stripe.transfers.create({
        amount: 70000,
        currency: "usd",
        destination: "acct_AGENCY",
        transfer_group: "booking_123"
      });

      await stripe.transfers.create({
        amount: 20000,
        currency: "usd",
        destination: "acct_SUPPLIER",
        transfer_group: "booking_123"
      });
    }




  //webhook index.js

    app.post('/webhook', express.raw({type: 'application/json'}), (req, res) => {

    const event = stripe.webhooks.constructEvent(
      req.body,
      req.headers['stripe-signature'],
      process.env.STRIPE_WEBHOOK_SECRET
    );

    if (event.type === 'payment_intent.succeeded') {
      console.log("Payment success")
    }

    res.send()
    });

