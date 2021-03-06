Hand Detection
==============

.. highlight:: cpp

HandDetector::PerPixRegression
------------------------------

.. ocv:class:: PerPixRegression

Derived class from HandDetector class. Detection method used in this class is pixel level hand detection as discussed by Kris Kitani and Cheng Li in their paper -

Pixel-level Hand Detection in Ego-Centric Videos - [KrisLi]_

Declaration::

    class CV_EXPORTS PerPixRegression : public HandDetector {
    public :
        struct CV_EXPORTS_W_SIMPLE Params {
            // specifies number of nearest neighbours for initialising the classifier
            CV_PROP_RW int knn;
            // specifies the number of number of nearest neighbours for classification
            CV_PROP_RW int numModels;
            // specifies the number of classifiers trained so far
            CV_PROP_RW int models;
            // specifies which features to use
            CV_PROP_RW string featureString;
            // specifies step size for considering pixels for processing - training
            CV_PROP_RW int training_step_size;
            // specifies step size for considering pixels for proccesing - testing/detecting
            CV_PROP_RW int testing_step_size;

            Params();

        };
        // constructor for PerPixRegression
        CV_WRAP PerPixRegression(const Params &parameters = Params());
        // Default destructor
        virtual ~PerPixRegression() { }

        // Function to train models on input image
        virtual bool train(Mat & _rgbImg, Mat & _depthImg, Mat & _mask, bool incremental = false);
        // Function to detect images - outputs binary image
        virtual void detect(Mat & _rgbImg, Mat & _depthImg, OutputArray probImg);
        // save trained models with general configuration file with configFileName, global feature files with featureFilePrefix, models with modelFilePrefix in that order in a vector. All names without .xml
        virtual bool save(vector<String> &fileNamePrefix);
        // load classifier from saved files - symmetric to load
        virtual bool load(vector<String> &fileNamePrefix);

    protected :
        int bs;
        Size sz;
        Params param;
        vector<int> indices;
        vector<float> dists;
        Mat                         descriptors;
        vector<KeyPoint>            kp;
        Mat                         responseAvg;
        Mat                         responseVec;
        Mat                         responseImg;
        Mat                         rawImg;
        Mat                         bluImg;
        Mat                         postProcessImg;               // post processed


        ///------------------///
        //classifier
        vector<CvRTrees*>           classifier;
        // search tree
        flann::Index                searchTree;
        // flann index params
        flann::IndexParams          indexParams;
        // feature extractor
        LcFeatureExtractor          extractor;
        // object storing hsv features for matching
        Mat                         histAll;              // do not destroy!
        // random tree params
        CvRTParams RTparams;

        // variables specifying intializing of modules
        bool flannInit;
        bool featureInit;
        bool classifierInit;

        /*------------------Member functions-----------------------------*/

        // Function for testing image (i.e detection)
        void test(Mat &img,int num_models, OutputArray probImg);
        // Function to convert output from vector of points to image
        Mat postprocess(Mat &img,vector<Point2f> &pt);

        // Function to compute color Histogram
        void computeColorHist_HSV(Mat &src, Mat &hist);
        // Funtion to raterise result vector
        void rasterizeResVec(Mat &img, Mat&res,vector<KeyPoint> &keypts, Size s);
        // initialize nearest neighbour search
        void initialiseFLANN(void);
    };

PerPixRegression::Params
------------------------

General structure for defining various parameters used to set up PerPixRegression.

    * **knn** – defines the number of nearest neighbour models searched to be used for detection. Nearest neighbours are defined in terms of similar HSV histograms. Refer research paper for more details.
    
        * default - 10
    
    * **numModels** – defines the number of models to be actually used for detection among the 'knn' nearest neighour.
    
        * default - 10
    
    * **models** – defines number of models currently trained.
    
    * **featureString** – defines the different features for feature extraction.
    
        * s – SIFT
        
        * h – HOG
        
        * l – LAB
        
        * v – HSV
        
        * b – BRIEF
        
        * o – ORB
        
        * r – RGB
        
        * u – SURF
        
      .. note:: default is “rvl” for {RGB, HSV, LAB} features. Refer paper/source code on how features are extracted.
    
    * **training_step_size** – defines the number of pixels to skip (1 – for no skipping) during training.
    
        * default - 3
    
    * **testing_step_size** – defines the number of pixels to skip (1 – for no skipping) during detection/testing.
    
        * default - 3
    

.. [KrisLi] Li, Cheng, and Kris M. Kitani. "Pixel-level Hand Detection in Ego-Centric Videos." This paper is available online at http://www.cs.cmu.edu/~kkitani/Publications_files/LK-CVPR2013.pdf
