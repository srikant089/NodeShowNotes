// MongoDB Models

    //User.Model.js
        import mongoose from "mongoose";
        const userSchema = new mongoose.Schema({
        name: String,
        email: { type: String, unique: true },
        password: String,
        isAdmin: { type: Boolean, default: false }
        }, { timestamps: true });

        export default mongoose.model("User", userSchema);


    //Brand.Model.js

        const brandSchema = new mongoose.Schema({
        name: { type: String, required: true }
        });

        export default mongoose.model("Brand", brandSchema);

    // Category.Model.js

        const categorySchema = new mongoose.Schema({
        name: { type: String, required: true }, // Men, Women, Kids
        });

        export default mongoose.model("Category", categorySchema);

    // Product.Model.js

        const productSchema = new mongoose.Schema({
        name: String,
        description: String,
        price: Number,
        brand: { type: mongoose.Schema.Types.ObjectId, ref: "Brand" },
        category: { type: mongoose.Schema.Types.ObjectId, ref: "Category" },

        variants: [
            {
                color: String,
                size: String,
                quantity: Number
            }
        ],

        images: [String],
        }, { timestamps: true });

        export default mongoose.model("Product", productSchema);


    //  Order.Model,JS

        const orderSchema = new mongoose.Schema({
            user: { type: mongoose.Schema.Types.ObjectId, ref: "User" },

            orderItems: [
                {
                product: { type: mongoose.Schema.Types.ObjectId, ref: "Product" },
                name: String,
                price: Number,
                quantity: Number,
                color: String,
                size: String
                }
            ],

            shippingAddress: {
                address: String,
                city: String,
                postalCode: String,
                country: String
            },

            totalPrice: Number,
            isPaid: Boolean,
            paidAt: Date,
        }, { timestamps: true });

        export default mongoose.model("Order", orderSchema);

    // Cart.Model.js

        const cartSchema = new mongoose.Schema({
        user: { type: mongoose.Schema.Types.ObjectId, ref: "User" },
        items: [
            {
            product: { type: mongoose.Schema.Types.ObjectId, ref: "Product" },
            color: String,
            size: String,
            quantity: Number
            }
        ],
        }, { timestamps: true });

        export default mongoose.model("Cart", cartSchema);