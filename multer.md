// backend/app.js

    const express = require('express');
    const bodyParser = require('body-parser');
    const dotenv = require('dotenv').config();

    const uploadImageRouter = require('./routes/uploadImageRouter');

    const app = express();
    app.use(express.json());
    app.use(bodyParser.urlencoded({ extended:false }));

    app.use('/images', express.static('public/images'));

    
    app.use('/uploadImage', uploadImageRouter);

    const PORT = process.env.PORT || 5000;
    app.listen(PORT, () => {
        console.log("Server Started: ", PORT)
    })

// backend/routes/uploadImageRouter.js

    const express = require('express');
    const upload = require('../middlewares/multer.js');

    const { uploadImage } = require('../controllers/uploadImageController.js');

    const router = express.Router();

    router.post('/uploadImage', upload.single("image"), uploadImage);

    module.exports = router;

// backend/middlewares/multer.js

    const multer  = require('multer');
    const path = require('path');

    // Set storage engine for multer
    const storage = multer.diskStorage({        
        destination: './public/images',
        filename: function(req, file, cb) {
            if(file.mimetype === 'image/jpeg' || file.mimetype === 'image/jpg' || file.mimetype === 'image/png'){
                const uniqueName = file.fieldname + '-' + Date.now() + path.extname(file.originalname)
                cb(null, uniqueName);
            }else{
                done('file type not supportes', false)
            }
        }
    });
      
    // Initialize multer upload
    const upload = multer({
        storage: storage,
        limits: { fileSize: 1024 * 1024 * 5 } // Limiting file size to 5MB
    });

    module.exports = upload;



// backend/controllers/uploadImageController.js
    const uploadImage = async (req,res) => {
        const  file: req.file
        res.status(200).json({file});

    }

    module.exports = {
        uploadImage,
    }