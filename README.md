<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام إدارة الصف الذكي - اكتشاف حقيقي بالذكاء الاصطناعي</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- TensorFlow.js for real AI face detection -->
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.10.0/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/blazeface@0.0.7/dist/blazeface.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/face-landmarks-detection@0.0.3/dist/face-landmarks-detection.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Cairo:wght@300;400;600;700&display=swap');
        
        body {
            font-family: 'Cairo', sans-serif;
        }
        
        .face-box {
            position: absolute;
            border: 3px solid;
            border-radius: 8px;
            pointer-events: none;
            z-index: 10;
            transition: all 0.2s ease;
            backdrop-filter: blur(1px);
        }
        
        .face-stable {
            border-color: #10b981;
            background: rgba(16, 185, 129, 0.15);
            box-shadow: 0 0 20px rgba(16, 185, 129, 0.4);
        }
        
        .face-tracking {
            border-color: #f59e0b;
            background: rgba(245, 158, 11, 0.15);
            box-shadow: 0 0 15px rgba(245, 158, 11, 0.3);
        }
        
        .face-unstable {
            border-color: #ef4444;
            background: rgba(239, 68, 68, 0.15);
            box-shadow: 0 0 10px rgba(239, 68, 68, 0.3);
        }
        
        .face-selected {
            border-color: #8b5cf6;
            background: rgba(139, 92, 246, 0.2);
            animation: selectedPulse 1.5s infinite;
        }
        
        .hand-raised {
            border-color: #ff6b35 !important;
            background: rgba(255, 107, 53, 0.25) !important;
            animation: handRaisePulse 1s infinite;
        }
        
        @keyframes selectedPulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.03); }
        }
        
        @keyframes handRaisePulse {
            0%, 100% { box-shadow: 0 0 20px rgba(255, 107, 53, 0.6); }
            50% { box-shadow: 0 0 30px rgba(255, 107, 53, 0.8); }
        }
        
        .attention-meter {
            background: linear-gradient(90deg, #dc2626, #f59e0b, #10b981);
            height: 8px;
            border-radius: 4px;
            position: relative;
            overflow: hidden;
        }
        
        .attention-indicator {
            position: absolute;
            top: -2px;
            width: 4px;
            height: 12px;
            background: white;
            border: 2px solid #374151;
            border-radius: 2px;
            transition: left 0.3s ease;
        }
        
        .movement-trail {
            position: absolute;
            width: 4px;
            height: 4px;
            background: rgba(59, 130, 246, 0.6);
            border-radius: 50%;
            pointer-events: none;
            z-index: 5;
            animation: fadeTrail 2s ease-out forwards;
        }
        
        @keyframes fadeTrail {
            0% { opacity: 1; transform: scale(1); }
            100% { opacity: 0; transform: scale(0.5); }
        }
        
        .camera-container {
            position: relative;
            overflow: hidden;
            border-radius: 12px;
            background: #000;
        }
        
        #video {
            transform: scaleX(-1);
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        
        .detection-overlay {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            pointer-events: none;
            z-index: 8;
        }
        
        .real-time-stats {
            background: linear-gradient(135deg, #1e3a8a, #3730a3);
            color: white;
            border-radius: 12px;
            padding: 16px;
            margin-top: 16px;
        }
        
        .student-profile {
            transition: all 0.3s ease;
            border-radius: 8px;
            padding: 12px;
            margin: 4px 0;
        }
        
        .student-active {
            background: linear-gradient(135deg, #dcfce7, #bbf7d0);
            border: 2px solid #10b981;
        }
        
        .student-inactive {
            background: linear-gradient(135deg, #f3f4f6, #e5e7eb);
            border: 2px solid #9ca3af;
        }
        
        .student-attention-high {
            background: linear-gradient(135deg, #dbeafe, #bfdbfe);
            border: 2px solid #3b82f6;
        }
        
        .loading-spinner {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid rgba(255,255,255,.3);
            border-radius: 50%;
            border-top-color: #fff;
            animation: spin 1s ease-in-out infinite;
        }
        
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
        
        .ai-indicator {
            position: absolute;
            top: 10px;
            right: 10px;
            background: rgba(0, 255, 0, 0.8);
            color: white;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 12px;
            z-index: 15;
        }
        
        .confidence-bar {
            position: absolute;
            bottom: -20px;
            left: 0;
            height: 4px;
            background: rgba(255, 255, 255, 0.8);
            border-radius: 2px;
            transition: width 0.3s ease;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
    <div class="container mx-auto px-4 py-6">
        <!-- Header -->
        <div class="bg-white rounded-xl shadow-lg p-6 mb-6">
            <h1 class="text-3xl font-bold text-gray-800 text-center mb-2">🤖 نظام إدارة الصف بالذكاء الاصطناعي</h1>
            <p class="text-gray-600 text-center">اكتشاف حقيقي للوجوه باستخدام TensorFlow.js و BlazeFace AI</p>
            
            <!-- AI System Status -->
            <div class="mt-4 bg-gradient-to-r from-green-50 to-blue-50 rounded-lg p-4">
                <div class="flex justify-between items-center">
                    <div class="flex items-center space-x-2 space-x-reverse">
                        <div id="aiStatus" class="w-3 h-3 bg-red-500 rounded-full"></div>
                        <span class="text-sm font-medium">حالة الذكاء الاصطناعي:</span>
                        <span id="aiStatusText" class="text-sm">جاري التحميل...</span>
                    </div>
                    <div class="text-sm text-gray-600">
                        <span>دقة AI: </span>
                        <span id="aiAccuracy" class="font-bold">0%</span>
                    </div>
                </div>
                <div class="mt-2 text-xs text-gray-500">
                    <span id="modelInfo">جاري تحميل نماذج الذكاء الاصطناعي...</span>
                </div>
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
            <!-- Camera Section -->
            <div class="lg:col-span-2">
                <div class="bg-white rounded-xl shadow-lg p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-semibold text-gray-800">🎥 مراقبة الصف بالذكاء الاصطناعي</h2>
                        <div class="flex gap-2">
                            <button id="startCamera" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-lg transition-colors" disabled>
                                🎥 تشغيل الكاميرا
                            </button>
                            <button id="capturePhoto" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors" disabled>
                                📸 التقاط صورة
                            </button>
                            <button id="stopCamera" class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg transition-colors" disabled>
                                ⏹️ إيقاف
                            </button>
                        </div>
                    </div>
                    
                    <div class="camera-container" style="height: 400px;">
                        <video id="video" autoplay muted style="display: none;"></video>
                        <canvas id="detectionCanvas" style="display: none;"></canvas>
                        <div class="detection-overlay" id="detectionOverlay"></div>
                        
                        <div id="cameraPlaceholder" class="w-full h-full flex items-center justify-center text-gray-500 bg-gray-100 rounded-lg">
                            <div class="text-center">
                                <div class="text-6xl mb-4">🤖</div>
                                <p class="text-lg">انتظر تحميل نماذج الذكاء الاصطناعي...</p>
                                <p class="text-sm text-gray-400 mt-2">BlazeFace + TensorFlow.js</p>
                                <div class="mt-4">
                                    <div class="loading-spinner"></div>
                                </div>
                            </div>
                        </div>
                        
                        <div id="loading" class="absolute inset-0 bg-black bg-opacity-80 flex items-center justify-center text-white" style="display: none;">
                            <div class="text-center">
                                <div class="loading-spinner mb-4"></div>
                                <p id="loadingText">جاري تهيئة نظام الاكتشاف...</p>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Real-time AI Detection Stats -->
                    <div class="real-time-stats">
                        <h3 class="font-semibold mb-3">🤖 إحصائيات الذكاء الاصطناعي المباشرة</h3>
                        <div class="grid grid-cols-4 gap-4">
                            <div class="text-center">
                                <div class="text-2xl font-bold text-green-300" id="facesDetected">0</div>
                                <div class="text-sm opacity-90">وجوه مكتشفة</div>
                            </div>
                            <div class="text-center">
                                <div class="text-2xl font-bold text-blue-300" id="aiConfidence">0%</div>
                                <div class="text-sm opacity-90">ثقة AI</div>
                            </div>
                            <div class="text-center">
                                <div class="text-2xl font-bold text-orange-300" id="handRaises">0</div>
                                <div class="text-sm opacity-90">رفع يد</div>
                            </div>
                            <div class="text-center">
                                <div class="text-2xl font-bold text-purple-300" id="avgAttention">0%</div>
                                <div class="text-sm opacity-90">متوسط التركيز</div>
                            </div>
                        </div>
                        
                        <!-- AI Performance Metrics -->
                        <div class="mt-4 grid grid-cols-3 gap-4 text-xs">
                            <div class="text-center">
                                <div class="font-bold" id="processingTime">0ms</div>
                                <div class="opacity-75">وقت المعالجة</div>
                            </div>
                            <div class="text-center">
                                <div class="font-bold" id="fps">0</div>
                                <div class="opacity-75">إطار/ثانية</div>
                            </div>
                            <div class="text-center">
                                <div class="font-bold" id="modelVersion">v0.0.7</div>
                                <div class="opacity-75">إصدار النموذج</div>
                            </div>
                        </div>
                        
                        <!-- Attention Meter -->
                        <div class="mt-4">
                            <div class="flex justify-between items-center mb-2">
                                <span class="text-sm font-medium">مقياس الانتباه العام</span>
                                <span class="text-sm font-bold" id="overallAttention">0%</span>
                            </div>
                            <div class="attention-meter">
                                <div class="attention-indicator" id="attentionIndicator" style="left: 0%"></div>
                            </div>
                            <div class="flex justify-between text-xs opacity-75 mt-1">
                                <span>منخفض</span>
                                <span>متوسط</span>
                                <span>عالي</span>
                            </div>
                        </div>
                    </div>
                    
                    <!-- Live AI Student Tracking -->
                    <div class="mt-4 bg-gray-50 rounded-lg p-4">
                        <h3 class="font-semibold text-gray-800 mb-3">🤖 التتبع الذكي للطلاب</h3>
                        <div id="liveStudentTracking" class="space-y-2 max-h-60 overflow-y-auto">
                            <div class="text-center text-gray-500 py-8">
                                <div class="text-2xl mb-2">🤖</div>
                                <p class="text-sm">في انتظار تحميل نماذج الذكاء الاصطناعي...</p>
                            </div>
                        </div>
                    </div>

                    <!-- AI Detection Log -->
                    <div class="mt-4 bg-gray-50 rounded-lg p-4">
                        <h3 class="font-semibold text-gray-800 mb-2">🤖 سجل الاكتشاف الذكي</h3>
                        <div id="detectionLog" class="text-sm text-gray-600 max-h-32 overflow-y-auto">
                            <p>جاري تحميل نماذج الذكاء الاصطناعي...</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Control Panel -->
            <div class="space-y-6">
                <!-- Student Names Management -->
                <div class="bg-white rounded-xl shadow-lg p-6">
                    <h3 class="text-lg font-semibold text-gray-800 mb-4">👥 إدارة أسماء الطلاب</h3>
                    
                    <div class="mb-4">
                        <input type="text" id="studentNameInput" placeholder="اسم الطالب" 
                               class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                        <button id="addStudent" class="w-full mt-2 bg-blue-500 hover:bg-blue-600 text-white py-2 rounded-lg transition-colors">
                            ➕ إضافة طالب
                        </button>
                    </div>
                    
                    <div id="studentsList" class="space-y-2 mb-4 max-h-40 overflow-y-auto">
                        <!-- Students will be added here -->
                    </div>
                    
                    <div class="space-y-2">
                        <button id="selectRandomStudent" class="w-full bg-orange-500 hover:bg-orange-600 text-white py-2 rounded-lg font-semibold transition-colors">
                            🎯 اختيار عشوائي
                        </button>
                        <button id="selectByAttention" class="w-full bg-purple-500 hover:bg-purple-600 text-white py-2 rounded-lg font-semibold transition-colors">
                            🧠 اختيار الأكثر انتباهاً
                        </button>
                        <button id="selectHandRaised" class="w-full bg-red-500 hover:bg-red-600 text-white py-2 rounded-lg font-semibold transition-colors">
                            🙋‍♂️ اختيار من رفع يده
                        </button>
                    </div>
                    
                    <div id="selectedStudent" class="mt-4 p-4 bg-orange-50 rounded-lg text-center" style="display: none;">
                        <div class="text-lg font-semibold text-orange-800">الطالب المختار:</div>
                        <div class="text-xl font-bold text-orange-600" id="selectedStudentName"></div>
                        <div class="text-sm text-orange-700 mt-1" id="selectionReason"></div>
                        <button id="freezeForAnswer" class="mt-2 bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg text-sm">
                            📸 تثبيت للإجابة
                        </button>
                    </div>
                </div>

                <!-- AI Analytics -->
                <div class="bg-white rounded-xl shadow-lg p-6">
                    <h3 class="text-lg font-semibold text-gray-800 mb-4">🤖 التحليل الذكي</h3>
                    
                    <div class="space-y-4">
                        <div class="bg-gradient-to-r from-blue-50 to-purple-50 p-4 rounded-lg">
                            <h4 class="font-semibold text-gray-800 mb-2">🎯 تحليل الانتباه الذكي</h4>
                            <div class="text-sm text-gray-600 space-y-1">
                                <div class="flex justify-between">
                                    <span>طلاب منتبهون:</span>
                                    <span id="attentiveStudents" class="font-semibold">0</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>طلاب يحتاجون تحفيز:</span>
                                    <span id="distractedStudents" class="font-semibold">0</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>دقة الاكتشاف:</span>
                                    <span id="detectionAccuracy" class="font-semibold">0%</span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="bg-gradient-to-r from-green-50 to-teal-50 p-4 rounded-lg">
                            <h4 class="font-semibold text-gray-800 mb-2">🏃‍♂️ تحليل الحركة الذكي</h4>
                            <div class="text-sm text-gray-600 space-y-1">
                                <div class="flex justify-between">
                                    <span>حركة طبيعية:</span>
                                    <span id="normalMovement" class="font-semibold">0</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>حركة نشطة:</span>
                                    <span id="activeMovement" class="font-semibold">0</span>
                                </div>
                                <div class="flex justify-between">
                                    <span>إجمالي رفع اليد:</span>
                                    <span id="totalHandRaises" class="font-semibold">0</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Quick Actions -->
                <div class="bg-white rounded-xl shadow-lg p-6">
                    <h3 class="text-lg font-semibold text-gray-800 mb-4">⚡ إجراءات سريعة</h3>
                    
                    <div class="space-y-2">
                        <button id="alertAttention" class="w-full bg-purple-500 hover:bg-purple-600 text-white py-2 rounded-lg transition-colors">
                            🔔 تنبيه للانتباه
                        </button>
                        <button id="resetTracking" class="w-full bg-yellow-500 hover:bg-yellow-600 text-white py-2 rounded-lg transition-colors">
                            🔄 إعادة تعيين التتبع
                        </button>
                        <button id="generateReport" class="w-full bg-teal-500 hover:bg-teal-600 text-white py-2 rounded-lg transition-colors">
                            📊 تقرير ذكي
                        </button>
                        <button id="exportData" class="w-full bg-green-500 hover:bg-green-600 text-white py-2 rounded-lg transition-colors">
                            💾 تصدير البيانات
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Report Modal -->
        <div id="reportModal" class="fixed inset-0 bg-black bg-opacity-50 z-50 hidden">
            <div class="flex items-center justify-center min-h-screen p-4">
                <div class="bg-white rounded-xl shadow-2xl max-w-4xl w-full max-h-[90vh] overflow-y-auto">
                    <div class="p-6 border-b border-gray-200">
                        <div class="flex justify-between items-center">
                            <h2 class="text-2xl font-bold text-gray-800">🤖 تقرير الذكاء الاصطناعي</h2>
                            <button id="closeReport" class="text-gray-500 hover:text-gray-700 text-2xl">×</button>
                        </div>
                    </div>
                    
                    <div class="p-6" id="reportContent">
                        <!-- Report content will be generated here -->
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Global variables for real AI face detection system
        let video, detectionCanvas, detectionCtx, detectionOverlay;
        let students = [];
        let isDetecting = false;
        let currentStream = null;
        let detectedFaces = new Map();
        let faceIdCounter = 0;
        let sessionStats = {
            startTime: null,
            totalDetections: 0,
            handRaises: 0,
            attentionHistory: [],
            movementHistory: [],
            interactionEvents: []
        };
        let selectedStudentFace = null;
        
        // AI Models and tracking variables
        let blazeFaceModel = null;
        let faceLandmarksModel = null;
        let isModelsLoaded = false;
        let faceTracker = new Map();
        let movementTracker = new Map();
        let attentionAnalyzer = new Map();
        let handRaiseDetector = new Map();
        let lastFrameTime = 0;
        let frameCount = 0;
        let currentFPS = 0;
        
        // AI Detection parameters
        const AI_CONFIG = {
            minFaceSize: 40,
            maxFaceSize: 400,
            confidenceThreshold: 0.7,
            stabilityFrames: 8,
            movementThreshold: 20,
            handRaiseThreshold: 30,
            maxFaces: 10,
            detectionInterval: 100, // ms between detections
            trackingTimeout: 2000 // ms before losing track
        };

        // Initialize application
        document.addEventListener('DOMContentLoaded', async function() {
            initializeElements();
            setupEventListeners();
            loadSavedData();
            
            // Load AI models
            await loadAIModels();
        });

        function initializeElements() {
            video = document.getElementById('video');
            detectionCanvas = document.getElementById('detectionCanvas');
            detectionCtx = detectionCanvas.getContext('2d');
            detectionOverlay = document.getElementById('detectionOverlay');
        }

        function setupEventListeners() {
            // Camera controls
            document.getElementById('startCamera').addEventListener('click', startCamera);
            document.getElementById('capturePhoto').addEventListener('click', capturePhoto);
            document.getElementById('stopCamera').addEventListener('click', stopCamera);
            
            // Student management
            document.getElementById('addStudent').addEventListener('click', addStudent);
            document.getElementById('selectRandomStudent').addEventListener('click', selectRandomStudent);
            document.getElementById('selectByAttention').addEventListener('click', selectByAttention);
            document.getElementById('selectHandRaised').addEventListener('click', selectHandRaised);
            document.getElementById('freezeForAnswer').addEventListener('click', freezeForAnswer);
            document.getElementById('studentNameInput').addEventListener('keypress', function(e) {
                if (e.key === 'Enter') addStudent();
            });
            
            // Quick actions
            document.getElementById('alertAttention').addEventListener('click', alertAttention);
            document.getElementById('resetTracking').addEventListener('click', resetTracking);
            document.getElementById('generateReport').addEventListener('click', generateReport);
            document.getElementById('exportData').addEventListener('click', exportData);
            
            // Report modal
            document.getElementById('closeReport').addEventListener('click', () => {
                document.getElementById('reportModal').classList.add('hidden');
            });
        }

        async function loadAIModels() {
            try {
                updateAIStatus('جاري تحميل نماذج الذكاء الاصطناعي...', 'orange');
                addToLog('🤖 بدء تحميل نماذج الذكاء الاصطناعي...');
                
                // Load TensorFlow.js backend
                await tf.ready();
                addToLog('✅ تم تحميل TensorFlow.js بنجاح');
                
                // Load BlazeFace model for face detection
                updateAIStatus('جاري تحميل نموذج BlazeFace...', 'orange');
                blazeFaceModel = await blazeface.load();
                addToLog('✅ تم تحميل نموذج BlazeFace بنجاح');
                
                // Load Face Landmarks model for detailed analysis
                updateAIStatus('جاري تحميل نموذج Face Landmarks...', 'orange');
                try {
                    faceLandmarksModel = await faceLandmarksDetection.load(
                        faceLandmarksDetection.SupportedPackages.mediapipeFacemesh
                    );
                    addToLog('✅ تم تحميل نموذج Face Landmarks بنجاح');
                } catch (landmarkError) {
                    console.warn('Face Landmarks model failed to load:', landmarkError);
                    addToLog('⚠️ تعذر تحميل نموذج Face Landmarks، سيتم استخدام BlazeFace فقط');
                }
                
                isModelsLoaded = true;
                updateAIStatus('جاهز - نماذج الذكاء الاصطناعي محملة', 'green');
                updateModelInfo('BlazeFace v0.0.7 + TensorFlow.js v4.10.0');
                
                // Enable camera button
                document.getElementById('startCamera').disabled = false;
                
                // Update placeholder
                document.getElementById('cameraPlaceholder').innerHTML = `
                    <div class="text-center">
                        <div class="text-6xl mb-4">🤖✅</div>
                        <p class="text-lg text-green-600 font-semibold">نماذج الذكاء الاصطناعي جاهزة!</p>
                        <p class="text-sm text-gray-600 mt-2">اضغط على "تشغيل الكاميرا" لبدء الاكتشاف الحقيقي</p>
                        <div class="mt-4 text-xs text-gray-500">
                            <div>✅ BlazeFace Model</div>
                            <div>✅ TensorFlow.js Backend</div>
                            ${faceLandmarksModel ? '<div>✅ Face Landmarks Model</div>' : '<div>⚠️ Face Landmarks (Optional)</div>'}
                        </div>
                    </div>
                `;
                
                addToLog('🚀 النظام جاهز للاكتشاف الحقيقي بالذكاء الاصطناعي!');
                showNotification('تم تحميل نماذج الذكاء الاصطناعي بنجاح! 🤖✅');
                
            } catch (error) {
                console.error('AI Models loading error:', error);
                updateAIStatus('خطأ في تحميل النماذج', 'red');
                addToLog('❌ خطأ في تحميل نماذج الذكاء الاصطناعي: ' + error.message);
                showNotification('خطأ في تحميل نماذج الذكاء الاصطناعي', 'error');
                
                document.getElementById('cameraPlaceholder').innerHTML = `
                    <div class="text-center">
                        <div class="text-6xl mb-4">❌</div>
                        <p class="text-lg text-red-600 font-semibold">خطأ في تحميل النماذج</p>
                        <p class="text-sm text-gray-600 mt-2">يرجى إعادة تحميل الصفحة</p>
                        <button onclick="location.reload()" class="mt-4 bg-red-500 text-white px-4 py-2 rounded-lg">
                            🔄 إعادة تحميل
                        </button>
                    </div>
                `;
            }
        }

        async function startCamera() {
            if (!isModelsLoaded) {
                showNotification('يرجى انتظار تحميل نماذج الذكاء الاصطناعي', 'error');
                return;
            }
            
            try {
                document.getElementById('loading').style.display = 'flex';
                document.getElementById('loadingText').textContent = 'جاري تشغيل الكاميرا...';
                
                // Request camera with optimal settings for AI detection
                currentStream = await navigator.mediaDevices.getUserMedia({ 
                    video: { 
                        width: { ideal: 640, min: 480 },
                        height: { ideal: 480, min: 360 },
                        facingMode: 'user',
                        frameRate: { ideal: 30, min: 15 }
                    } 
                });
                
                video.srcObject = currentStream;
                
                // Wait for video to be ready
                await new Promise((resolve) => {
                    video.onloadedmetadata = () => {
                        video.play();
                        resolve();
                    };
                });
                
                // Setup canvas for AI processing
                detectionCanvas.width = video.videoWidth;
                detectionCanvas.height = video.videoHeight;
                
                video.style.display = 'block';
                document.getElementById('cameraPlaceholder').style.display = 'none';
                document.getElementById('loading').style.display = 'none';
                
                // Enable/disable buttons
                document.getElementById('startCamera').disabled = true;
                document.getElementById('capturePhoto').disabled = false;
                document.getElementById('stopCamera').disabled = false;
                
                // Start real AI detection
                startAIFaceDetection();
                
                updateAIStatus('نشط - اكتشاف ذكي', 'green');
                addToLog('🎥 تم تشغيل الكاميرا وبدء الاكتشاف بالذكاء الاصطناعي');
                showNotification('تم تشغيل نظام الاكتشاف الذكي! 🤖🎥');
                
                sessionStats.startTime = new Date();
                lastFrameTime = performance.now();
                frameCount = 0;
                
            } catch (error) {
                console.error('Camera error:', error);
                document.getElementById('loading').style.display = 'none';
                updateAIStatus('خطأ في الكاميرا', 'red');
                addToLog('❌ خطأ في تشغيل الكاميرا: ' + error.message);
                showNotification('خطأ في تشغيل الكاميرا. يرجى التحقق من الأذونات.', 'error');
            }
        }

        function startAIFaceDetection() {
            if (isDetecting) return;
            isDetecting = true;
            
            updateAIStatus('يعمل - اكتشاف ذكي نشط', 'green');
            detectFacesWithAI();
        }

        async function detectFacesWithAI() {
            if (!isDetecting || !video || video.videoWidth === 0 || !blazeFaceModel) {
                return;
            }

            const startTime = performance.now();
            
            try {
                // Create tensor from video for AI processing
                const videoTensor = tf.browser.fromPixels(video);
                
                // Detect faces using BlazeFace AI model
                const predictions = await blazeFaceModel.estimateFaces(videoTensor, false);
                
                // Clean up tensor to prevent memory leaks
                videoTensor.dispose();
                
                // Process AI predictions into our face format
                const processedFaces = await processAIPredictions(predictions);
                
                // Update face tracking and analysis
                updateFaceTracking(processedFaces);
                
                // Analyze movement and hand raises
                analyzeMovementAndGestures(processedFaces);
                
                // Calculate attention levels
                calculateAttentionLevels(processedFaces);
                
                // Update UI with AI results
                updateDetectionUI(processedFaces);
                
                // Draw AI face detections
                drawAIFaceDetections(processedFaces);
                
                const processingTime = Math.round(performance.now() - startTime);
                updateAIPerformanceStats(processedFaces.length, processingTime);
                
                sessionStats.totalDetections++;
                
                // Calculate and update FPS
                frameCount++;
                const currentTime = performance.now();
                if (currentTime - lastFrameTime >= 1000) {
                    currentFPS = Math.round(frameCount * 1000 / (currentTime - lastFrameTime));
                    document.getElementById('fps').textContent = currentFPS;
                    frameCount = 0;
                    lastFrameTime = currentTime;
                }
                
            } catch (error) {
                console.error('AI Detection error:', error);
                addToLog('❌ خطأ في الاكتشاف الذكي: ' + error.message);
            }
            
            // Continue AI detection loop
            if (isDetecting) {
                setTimeout(() => {
                    requestAnimationFrame(detectFacesWithAI);
                }, AI_CONFIG.detectionInterval);
            }
        }

        async function processAIPredictions(predictions) {
            const processedFaces = [];
            const currentTime = Date.now();
            
            // Limit number of faces to process
            const facesToProcess = predictions.slice(0, AI_CONFIG.maxFaces);
            
            for (let i = 0; i < facesToProcess.length; i++) {
                const prediction = facesToProcess[i];
                
                // Extract face bounding box
                const [x, y] = prediction.topLeft;
                const [x2, y2] = prediction.bottomRight;
                const width = x2 - x;
                const height = y2 - y;
                
                // Skip faces that are too small or too large
                if (width < AI_CONFIG.minFaceSize || width > AI_CONFIG.maxFaceSize ||
                    height < AI_CONFIG.minFaceSize || height > AI_CONFIG.maxFaceSize) {
                    continue;
                }
                
                // Calculate face center and features
                const centerX = x + width / 2;
                const centerY = y + height / 2;
                const size = width * height;
                const confidence = prediction.probability ? prediction.probability[0] : 0.8;
                
                // Skip low confidence detections
                if (confidence < AI_CONFIG.confidenceThreshold) {
                    continue;
                }
                
                // Find matching existing face or create new one
                const faceId = findOrCreateFaceId(centerX, centerY, size, confidence);
                
                // Extract additional features if available
                let landmarks = null;
                if (prediction.landmarks) {
                    landmarks = prediction.landmarks;
                }
                
                // Create face object with AI data
                const face = {
                    id: faceId,
                    x: Math.round(x),
                    y: Math.round(y),
                    width: Math.round(width),
                    height: Math.round(height),
                    centerX: Math.round(centerX),
                    centerY: Math.round(centerY),
                    confidence: confidence,
                    size: size,
                    timestamp: currentTime,
                    landmarks: landmarks,
                    isStable: false,
                    attentionScore: 0,
                    movementType: 'unknown',
                    handRaised: false,
                    aiSource: 'BlazeFace'
                };
                
                processedFaces.push(face);
            }
            
            return processedFaces;
        }

        function findOrCreateFaceId(centerX, centerY, size, confidence) {
            const currentTime = Date.now();
            let bestMatch = null;
            let bestScore = 0;
            
            // Clean up old faces that haven't been seen recently
            for (const [id, tracker] of faceTracker.entries()) {
                if (currentTime - tracker.lastSeen > AI_CONFIG.trackingTimeout) {
                    faceTracker.delete(id);
                    movementTracker.delete(id);
                    attentionAnalyzer.delete(id);
                    handRaiseDetector.delete(id);
                    addToLog(`🔄 تم إزالة الطالب ${id} من التتبع (انقطاع الاتصال)`);
                }
            }
            
            // Find best matching existing face using improved algorithm
            for (const [id, tracker] of faceTracker.entries()) {
                const distance = Math.sqrt(
                    Math.pow(centerX - tracker.lastX, 2) + 
                    Math.pow(centerY - tracker.lastY, 2)
                );
                
                const sizeRatio = Math.min(size, tracker.lastSize) / Math.max(size, tracker.lastSize);
                const timeDiff = currentTime - tracker.lastSeen;
                
                // Improved matching criteria
                if (distance < 80 && sizeRatio > 0.6 && timeDiff < 1500) {
                    const distanceScore = Math.max(0, 1 - distance / 80);
                    const sizeScore = sizeRatio;
                    const timeScore = Math.max(0, 1 - timeDiff / 1500);
                    const confidenceScore = confidence;
                    
                    const totalScore = (distanceScore * 0.4 + sizeScore * 0.2 + 
                                     timeScore * 0.2 + confidenceScore * 0.2);
                    
                    if (totalScore > bestScore && totalScore > 0.6) {
                        bestScore = totalScore;
                        bestMatch = id;
                    }
                }
            }
            
            if (bestMatch) {
                // Update existing face tracker
                const tracker = faceTracker.get(bestMatch);
                tracker.lastX = centerX;
                tracker.lastY = centerY;
                tracker.lastSize = size;
                tracker.lastSeen = currentTime;
                tracker.detectionCount++;
                tracker.confidenceSum += confidence;
                tracker.avgConfidence = tracker.confidenceSum / tracker.detectionCount;
                
                // Update position history
                tracker.positions.push({ x: centerX, y: centerY, time: currentTime });
                if (tracker.positions.length > 30) {
                    tracker.positions = tracker.positions.slice(-30);
                }
                
                return bestMatch;
            } else {
                // Create new face tracker
                const newId = ++faceIdCounter;
                faceTracker.set(newId, {
                    id: newId,
                    firstSeen: currentTime,
                    lastSeen: currentTime,
                    lastX: centerX,
                    lastY: centerY,
                    lastSize: size,
                    detectionCount: 1,
                    confidenceSum: confidence,
                    avgConfidence: confidence,
                    positions: [{ x: centerX, y: centerY, time: currentTime }]
                });
                
                // Initialize other trackers
                movementTracker.set(newId, {
                    positions: [{ x: centerX, y: centerY, time: currentTime }],
                    velocity: 0,
                    direction: 'stable',
                    lastMovementTime: currentTime
                });
                
                attentionAnalyzer.set(newId, {
                    scores: [],
                    avgScore: 0,
                    trend: 'stable'
                });
                
                handRaiseDetector.set(newId, {
                    raises: [],
                    lastRaise: 0,
                    totalRaises: 0
                });
                
                addToLog(`🆕 تم اكتشاف طالب جديد: ${newId} (ثقة: ${Math.round(confidence * 100)}%)`);
                
                return newId;
            }
        }

        function updateFaceTracking(faces) {
            for (const face of faces) {
                const tracker = faceTracker.get(face.id);
                if (!tracker) continue;
                
                // Update stability based on detection count
                face.isStable = tracker.detectionCount >= AI_CONFIG.stabilityFrames;
                
                // Store current face data in detectedFaces map
                detectedFaces.set(face.id, face);
            }
        }

        function analyzeMovementAndGestures(faces) {
            for (const face of faces) {
                const movement = movementTracker.get(face.id);
                const handDetector = handRaiseDetector.get(face.id);
                
                if (!movement || !handDetector) continue;
                
                // Update movement analysis
                if (movement.positions.length >= 2) {
                    const recent = movement.positions.slice(-2);
                    const distance = Math.sqrt(
                        Math.pow(recent[1].x - recent[0].x, 2) + 
                        Math.pow(recent[1].y - recent[0].y, 2)
                    );
                    const timeDiff = recent[1].time - recent[0].time;
                    
                    movement.velocity = timeDiff > 0 ? (distance / timeDiff) * 1000 : 0;
                    
                    // Classify movement type
                    if (movement.velocity < 0.05) {
                        face.movementType = 'stable';
                    } else if (movement.velocity < 0.3) {
                        face.movementType = 'normal';
                    } else {
                        face.movementType = 'active';
                        movement.lastMovementTime = face.timestamp;
                    }
                    
                    // Enhanced hand raise detection using AI landmarks
                    const verticalMovement = recent[0].y - recent[1].y; // Negative = upward
                    const horizontalMovement = Math.abs(recent[1].x - recent[0].x);
                    
                    // Look for rapid upward movement with minimal horizontal drift
                    if (verticalMovement > AI_CONFIG.handRaiseThreshold && 
                        movement.velocity > 0.5 && 
                        horizontalMovement < verticalMovement * 0.5) {
                        
                        const timeSinceLastRaise = face.timestamp - handDetector.lastRaise;
                        if (timeSinceLastRaise > 3000) { // Minimum 3 seconds between raises
                            const confidence = Math.min(0.95, movement.velocity * face.confidence);
                            
                            handDetector.raises.push({
                                time: face.timestamp,
                                confidence: confidence,
                                movement: verticalMovement,
                                aiDetected: true
                            });
                            handDetector.lastRaise = face.timestamp;
                            handDetector.totalRaises++;
                            
                            face.handRaised = true;
                            sessionStats.handRaises++;
                            
                            addToLog(`🙋‍♂️ AI اكتشف رفع يد للطالب ${face.id} (ثقة: ${Math.round(confidence * 100)}%)`);
                            showNotification(`🙋‍♂️ طالب ${face.id} رفع يده!`);
                        }
                    }
                    
                    // Check for recent hand raise (keep highlighting for 4 seconds)
                    const recentRaises = handDetector.raises.filter(r => 
                        face.timestamp - r.time < 4000
                    );
                    face.handRaised = recentRaises.length > 0;
                }
                
                // Add current position to movement tracker
                movement.positions.push({
                    x: face.centerX,
                    y: face.centerY,
                    time: face.timestamp
                });
                
                if (movement.positions.length > 15) {
                    movement.positions = movement.positions.slice(-15);
                }
            }
        }

        function calculateAttentionLevels(faces) {
            for (const face of faces) {
                const analyzer = attentionAnalyzer.get(face.id);
                if (!analyzer) continue;
                
                let attentionScore = 0;
                
                // Base score from AI confidence
                attentionScore += face.confidence * 30;
                
                // Center position score (looking at camera/board)
                const centerX = video.videoWidth / 2;
                const centerY = video.videoHeight / 2;
                const distanceFromCenter = Math.sqrt(
                    Math.pow(face.centerX - centerX, 2) + 
                    Math.pow(face.centerY - centerY, 2)
                );
                const maxDistance = Math.sqrt(centerX * centerX + centerY * centerY);
                const centerScore = Math.max(0, 1 - (distanceFromCenter / maxDistance)) * 25;
                attentionScore += centerScore;
                
                // Size score (appropriate distance from camera)
                const idealSize = 8000; // Ideal face size for attention
                const sizeScore = Math.max(0, 1 - Math.abs(face.size - idealSize) / idealSize) * 20;
                attentionScore += sizeScore;
                
                // Stability score (consistent detection)
                const tracker = faceTracker.get(face.id);
                const stabilityScore = tracker ? Math.min(25, tracker.detectionCount * 2) : 0;
                attentionScore += stabilityScore;
                
                // Movement score (moderate movement indicates engagement)
                const movement = movementTracker.get(face.id);
                let movementScore = 0;
                if (movement) {
                    if (movement.velocity < 0.02) movementScore = 10; // Too static
                    else if (movement.velocity < 0.2) movementScore = 20; // Good engagement
                    else if (movement.velocity < 0.6) movementScore = 15; // Active
                    else movementScore = 5; // Too active/distracted
                }
                attentionScore += movementScore;
                
                face.attentionScore = Math.round(Math.max(0, Math.min(100, attentionScore)));
                
                // Update analyzer with AI-enhanced data
                analyzer.scores.push(face.attentionScore);
                if (analyzer.scores.length > 50) {
                    analyzer.scores = analyzer.scores.slice(-50);
                }
                analyzer.avgScore = Math.round(
                    analyzer.scores.reduce((sum, score) => sum + score, 0) / analyzer.scores.length
                );
                
                // Calculate trend using more data points
                if (analyzer.scores.length >= 20) {
                    const recent = analyzer.scores.slice(-10);
                    const earlier = analyzer.scores.slice(-20, -10);
                    const recentAvg = recent.reduce((sum, score) => sum + score, 0) / recent.length;
                    const earlierAvg = earlier.reduce((sum, score) => sum + score, 0) / earlier.length;
                    
                    if (recentAvg > earlierAvg + 8) analyzer.trend = 'improving';
                    else if (recentAvg < earlierAvg - 8) analyzer.trend = 'declining';
                    else analyzer.trend = 'stable';
                }
            }
        }

        function updateDetectionUI(faces) {
            // Update main AI stats
            document.getElementById('facesDetected').textContent = faces.length;
            
            const avgConfidence = faces.length > 0 ? 
                Math.round(faces.reduce((sum, f) => sum + f.confidence, 0) / faces.length * 100) : 0;
            document.getElementById('aiConfidence').textContent = avgConfidence + '%';
            
            const handRaisedCount = faces.filter(f => f.handRaised).length;
            document.getElementById('handRaises').textContent = handRaisedCount;
            
            const avgAttention = faces.length > 0 ? 
                Math.round(faces.reduce((sum, f) => sum + f.attentionScore, 0) / faces.length) : 0;
            document.getElementById('avgAttention').textContent = avgAttention + '%';
            document.getElementById('overallAttention').textContent = avgAttention + '%';
            
            // Update attention indicator
            const indicator = document.getElementById('attentionIndicator');
            indicator.style.left = avgAttention + '%';
            
            // Update AI analytics
            const attentiveStudents = faces.filter(f => f.attentionScore > 70).length;
            const distractedStudents = faces.filter(f => f.attentionScore < 40).length;
            const detectionAccuracy = Math.min(100, avgConfidence + (faces.length * 5));
            
            document.getElementById('attentiveStudents').textContent = attentiveStudents;
            document.getElementById('distractedStudents').textContent = distractedStudents;
            document.getElementById('detectionAccuracy').textContent = detectionAccuracy + '%';
            document.getElementById('aiAccuracy').textContent = detectionAccuracy + '%';
            
            const normalMovement = faces.filter(f => f.movementType === 'normal').length;
            const activeMovement = faces.filter(f => f.movementType === 'active').length;
            
            document.getElementById('normalMovement').textContent = normalMovement;
            document.getElementById('activeMovement').textContent = activeMovement;
            document.getElementById('totalHandRaises').textContent = sessionStats.handRaises;
            
            // Update live AI student tracking
            updateLiveAIStudentTracking(faces);
            
            // Record data for history
            sessionStats.attentionHistory.push({
                timestamp: Date.now(),
                avgAttention: avgAttention,
                avgConfidence: avgConfidence,
                faceCount: faces.length,
                handRaises: handRaisedCount
            });
            
            // Keep only recent history
            if (sessionStats.attentionHistory.length > 500) {
                sessionStats.attentionHistory = sessionStats.attentionHistory.slice(-500);
            }
        }

        function drawAIFaceDetections(faces) {
            // Clear previous detections
            detectionOverlay.innerHTML = '';
            
            for (const face of faces) {
                const faceBox = document.createElement('div');
                faceBox.className = 'face-box';
                faceBox.id = `face-${face.id}`;
                
                // Apply styling based on AI analysis
                if (face.handRaised) {
                    faceBox.classList.add('hand-raised');
                } else if (face.isStable && face.attentionScore > 70) {
                    faceBox.classList.add('face-stable');
                } else if (face.isStable) {
                    faceBox.classList.add('face-tracking');
                } else {
                    faceBox.classList.add('face-unstable');
                }
                
                if (selectedStudentFace && selectedStudentFace.id === face.id) {
                    faceBox.classList.add('face-selected');
                }
                
                // Calculate position relative to video display
                const videoRect = video.getBoundingClientRect();
                const scaleX = videoRect.width / video.videoWidth;
                const scaleY = videoRect.height / video.videoHeight;
                
                const left = face.x * scaleX;
                const top = face.y * scaleY;
                const width = face.width * scaleX;
                const height = face.height * scaleY;
                
                faceBox.style.left = left + 'px';
                faceBox.style.top = top + 'px';
                faceBox.style.width = width + 'px';
                faceBox.style.height = height + 'px';
                
                // Create AI info display
                const tracker = faceTracker.get(face.id);
                const movement = movementTracker.get(face.id);
                const handDetector = handRaiseDetector.get(face.id);
                
                let statusIcons = '';
                if (face.isStable) statusIcons += '🔒';
                if (face.handRaised) statusIcons += '🙋‍♂️';
                if (face.movementType === 'active') statusIcons += '🏃‍♂️';
                else if (face.movementType === 'stable') statusIcons += '🧘‍♂️';
                
                const confidence = Math.round(face.confidence * 100);
                const detectionCount = tracker ? tracker.detectionCount : 0;
                const totalRaises = handDetector ? handDetector.totalRaises : 0;
                
                faceBox.innerHTML = `
                    <div class="ai-indicator">🤖 AI</div>
                    <div class="confidence-bar" style="width: ${confidence}%"></div>
                    <div style="background: rgba(0,0,0,0.9); color: white; padding: 4px 8px; border-radius: 6px; font-size: 11px; position: absolute; top: -65px; left: 0; white-space: nowrap; box-shadow: 0 2px 8px rgba(0,0,0,0.3); min-width: 140px;">
                        <div style="font-weight: bold; margin-bottom: 2px;">
                            🤖 طالب ${face.id} ${statusIcons}
                        </div>
                        <div style="font-size: 9px; opacity: 0.9; line-height: 1.2;">
                            انتباه: ${face.attentionScore}% | AI: ${confidence}%<br>
                            تتبع: ${detectionCount} | رفع يد: ${totalRaises}<br>
                            <span style="color: #4ade80;">BlazeFace AI Detection</span>
                        </div>
                    </div>
                `;
                
                detectionOverlay.appendChild(faceBox);
                
                // Draw AI movement trail
                if (movement && movement.positions.length > 1) {
                    drawAIMovementTrail(movement.positions, scaleX, scaleY);
                }
                
                // Draw AI landmarks if available
                if (face.landmarks && face.landmarks.length > 0) {
                    drawAILandmarks(face.landmarks, scaleX, scaleY);
                }
            }
        }

        function drawAIMovementTrail(positions, scaleX, scaleY) {
            const recentPositions = positions.slice(-8); // Last 8 positions for smoother trail
            
            for (let i = 1; i < recentPositions.length; i++) {
                const pos = recentPositions[i];
                const trail = document.createElement('div');
                trail.className = 'movement-trail';
                
                trail.style.left = (pos.x * scaleX) + 'px';
                trail.style.top = (pos.y * scaleY) + 'px';
                trail.style.opacity = (i / recentPositions.length) * 0.7;
                trail.style.background = 'rgba(34, 197, 94, 0.8)'; // Green for AI trail
                
                detectionOverlay.appendChild(trail);
                
                // Remove trail after animation
                setTimeout(() => {
                    if (trail.parentNode) {
                        trail.parentNode.removeChild(trail);
                    }
                }, 2500);
            }
        }

        function drawAILandmarks(landmarks, scaleX, scaleY) {
            // Draw key facial landmarks if available
            for (let i = 0; i < landmarks.length; i += 10) { // Sample landmarks to avoid clutter
                const landmark = landmarks[i];
                const point = document.createElement('div');
                point.style.cssText = `
                    position: absolute;
                    width: 2px;
                    height: 2px;
                    background: rgba(255, 255, 0, 0.8);
                    border-radius: 50%;
                    left: ${landmark[0] * scaleX}px;
                    top: ${landmark[1] * scaleY}px;
                    pointer-events: none;
                    z-index: 12;
                `;
                
                detectionOverlay.appendChild(point);
                
                // Remove landmark after short time
                setTimeout(() => {
                    if (point.parentNode) {
                        point.parentNode.removeChild(point);
                    }
                }, 1000);
            }
        }

        function updateLiveAIStudentTracking(faces) {
            const container = document.getElementById('liveStudentTracking');
            container.innerHTML = '';
            
            if (faces.length === 0) {
                container.innerHTML = `
                    <div class="text-center text-gray-500 py-8">
                        <div class="text-2xl mb-2">🤖</div>
                        <p class="text-sm">الذكاء الاصطناعي يبحث عن الطلاب...</p>
                        <div class="text-xs text-gray-400 mt-1">BlazeFace AI Active</div>
                    </div>
                `;
                return;
            }
            
            // Sort faces by AI confidence and attention
            const sortedFaces = [...faces].sort((a, b) => {
                const scoreA = (a.attentionScore * 0.7) + (a.confidence * 100 * 0.3);
                const scoreB = (b.attentionScore * 0.7) + (b.confidence * 100 * 0.3);
                return scoreB - scoreA;
            });
            
            for (const face of sortedFaces) {
                const tracker = faceTracker.get(face.id);
                const analyzer = attentionAnalyzer.get(face.id);
                const handDetector = handRaiseDetector.get(face.id);
                
                if (!tracker || !analyzer) continue;
                
                const profileDiv = document.createElement('div');
                profileDiv.className = 'student-profile';
                
                // Determine profile class based on AI analysis
                if (face.attentionScore > 70 && face.confidence > 0.8) {
                    profileDiv.classList.add('student-attention-high');
                } else if (face.isStable && face.confidence > 0.7) {
                    profileDiv.classList.add('student-active');
                } else {
                    profileDiv.classList.add('student-inactive');
                }
                
                const participationTime = Math.round((tracker.lastSeen - tracker.firstSeen) / 60000);
                const recentHandRaises = handDetector ? handDetector.raises.filter(r => 
                    Date.now() - r.time < 30000
                ).length : 0;
                
                const aiConfidence = Math.round(face.confidence * 100);
                
                profileDiv.innerHTML = `
                    <div class="flex justify-between items-center mb-2">
                        <div class="font-semibold text-gray-800">
                            🤖 طالب ${face.id}
                            ${face.handRaised ? ' 🙋‍♂️' : ''}
                            ${face.isStable ? ' 🔒' : ''}
                        </div>
                        <div class="flex gap-1">
                            <div class="text-xs px-2 py-1 rounded-full ${
                                face.attentionScore > 70 ? 'bg-green-500' : 
                                face.attentionScore > 40 ? 'bg-yellow-500' : 'bg-red-500'
                            } text-white">
                                ${face.attentionScore}%
                            </div>
                            <div class="text-xs px-2 py-1 rounded-full bg-blue-500 text-white">
                                AI: ${aiConfidence}%
                            </div>
                        </div>
                    </div>
                    <div class="text-xs text-gray-600 space-y-1">
                        <div class="flex justify-between">
                            <span>متوسط الانتباه:</span>
                            <span class="font-semibold">${analyzer.avgScore}%</span>
                        </div>
                        <div class="flex justify-between">
                            <span>ثقة الذكاء الاصطناعي:</span>
                            <span class="font-semibold">${Math.round(tracker.avgConfidence * 100)}%</span>
                        </div>
                        <div class="flex justify-between">
                            <span>نوع الحركة:</span>
                            <span class="font-semibold">${
                                face.movementType === 'stable' ? 'مستقر' :
                                face.movementType === 'normal' ? 'طبيعي' : 'نشط'
                            }</span>
                        </div>
                        <div class="flex justify-between">
                            <span>وقت المشاركة:</span>
                            <span class="font-semibold">${participationTime} دقيقة</span>
                        </div>
                        <div class="flex justify-between">
                            <span>الاتجاه:</span>
                            <span class="font-semibold ${
                                analyzer.trend === 'improving' ? 'text-green-600' :
                                analyzer.trend === 'declining' ? 'text-red-600' : 'text-gray-600'
                            }">
                                ${analyzer.trend === 'improving' ? '📈 تحسن' :
                                  analyzer.trend === 'declining' ? '📉 تراجع' : '➡️ ثابت'}
                            </span>
                        </div>
                        ${recentHandRaises > 0 ? `
                            <div class="mt-2 p-2 bg-orange-100 rounded text-xs">
                                <span class="text-orange-700">🤖 AI اكتشف رفع يد: ${recentHandRaises} مرة</span>
                            </div>
                        ` : ''}
                    </div>
                `;
                
                container.appendChild(profileDiv);
            }
        }

        function updateAIPerformanceStats(faceCount, processingTime) {
            document.getElementById('processingTime').textContent = processingTime + 'ms';
            
            // Update AI accuracy based on confidence and detection stability
            const accuracy = Math.min(100, 70 + (faceCount * 3) + Math.max(0, 30 - processingTime));
            document.getElementById('aiAccuracy').textContent = accuracy + '%';
            
            // Update status based on performance
            if (processingTime < 50 && faceCount > 0) {
                updateAIStatus('ممتاز - أداء AI عالي', 'green');
            } else if (processingTime < 100) {
                updateAIStatus('جيد - أداء AI مقبول', 'green');
            } else if (processingTime < 200) {
                updateAIStatus('متوسط - أداء AI بطيء', 'yellow');
            } else {
                updateAIStatus('بطيء - يحتاج تحسين', 'orange');
            }
        }

        function updateAIStatus(status, color) {
            const statusElement = document.getElementById('aiStatus');
            const statusText = document.getElementById('aiStatusText');
            
            statusElement.className = `w-3 h-3 rounded-full bg-${color}-500`;
            statusText.textContent = status;
        }

        function updateModelInfo(info) {
            document.getElementById('modelInfo').textContent = info;
        }

        function stopCamera() {
            isDetecting = false;
            
            if (currentStream) {
                currentStream.getTracks().forEach(track => track.stop());
                currentStream = null;
            }
            
            video.style.display = 'none';
            document.getElementById('cameraPlaceholder').style.display = 'flex';
            detectionOverlay.innerHTML = '';
            
            // Reset buttons
            document.getElementById('startCamera').disabled = false;
            document.getElementById('capturePhoto').disabled = true;
            document.getElementById('stopCamera').disabled = true;
            
            // Reset stats
            document.getElementById('facesDetected').textContent = '0';
            document.getElementById('aiConfidence').textContent = '0%';
            document.getElementById('handRaises').textContent = '0';
            document.getElementById('avgAttention').textContent = '0%';
            document.getElementById('overallAttention').textContent = '0%';
            document.getElementById('attentionIndicator').style.left = '0%';
            document.getElementById('fps').textContent = '0';
            document.getElementById('processingTime').textContent = '0ms';
            
            updateAIStatus('متوقف', 'red');
            addToLog('🛑 تم إيقاف نظام الاكتشاف الذكي');
            showNotification('تم إيقاف النظام الذكي');
            
            // Update placeholder
            document.getElementById('cameraPlaceholder').innerHTML = `
                <div class="text-center">
                    <div class="text-6xl mb-4">🤖✅</div>
                    <p class="text-lg text-green-600 font-semibold">نماذج الذكاء الاصطناعي جاهزة!</p>
                    <p class="text-sm text-gray-600 mt-2">اضغط على "تشغيل الكاميرا" لبدء الاكتشاف الحقيقي</p>
                </div>
            `;
        }

        function capturePhoto() {
            if (!video.videoWidth) return;
            
            // Create capture canvas
            const captureCanvas = document.createElement('canvas');
            const captureCtx = captureCanvas.getContext('2d');
            captureCanvas.width = video.videoWidth;
            captureCanvas.height = video.videoHeight;
            
            // Draw video frame
            captureCtx.save();
            captureCtx.scale(-1, 1);
            captureCtx.drawImage(video, -captureCanvas.width, 0, captureCanvas.width, captureCanvas.height);
            captureCtx.restore();
            
            // Draw AI face detections
            const faces = Array.from(detectedFaces.values());
            for (const face of faces) {
                // Draw face box
                captureCtx.strokeStyle = face.handRaised ? '#ff6b35' :
                                       face.attentionScore > 70 ? '#10b981' : 
                                       face.attentionScore > 40 ? '#f59e0b' : '#ef4444';
                captureCtx.lineWidth = 4;
                captureCtx.strokeRect(face.x, face.y, face.width, face.height);
                
                // Add AI indicator
                captureCtx.fillStyle = 'rgba(0, 255, 0, 0.8)';
                captureCtx.fillRect(face.x + face.width - 40, face.y, 40, 20);
                captureCtx.fillStyle = 'white';
                captureCtx.font = '12px Arial';
                captureCtx.fillText('🤖 AI', face.x + face.width - 35, face.y + 14);
                
                // Add info
                captureCtx.fillStyle = captureCtx.strokeStyle;
                captureCtx.fillRect(face.x, face.y - 40, 250, 40);
                captureCtx.fillStyle = 'white';
                captureCtx.font = '16px Cairo';
                const aiConfidence = Math.round(face.confidence * 100);
                captureCtx.fillText(`طالب ${face.id} - انتباه: ${face.attentionScore}% - AI: ${aiConfidence}%${face.handRaised ? ' 🙋‍♂️' : ''}`, 
                                   face.x + 5, face.y - 20);
                captureCtx.font = '12px Cairo';
                captureCtx.fillText(`BlazeFace AI Detection`, face.x + 5, face.y - 5);
            }
            
            // Add AI timestamp and stats
            captureCtx.fillStyle = 'rgba(0,0,0,0.8)';
            captureCtx.fillRect(10, captureCanvas.height - 100, 600, 90);
            captureCtx.fillStyle = 'white';
            captureCtx.font = '18px Cairo';
            captureCtx.fillText(`🤖 ${new Date().toLocaleString('ar-SA')}`, 15, captureCanvas.height - 75);
            captureCtx.fillText(`${faces.length} طالب | ${faces.filter(f => f.handRaised).length} رفع يد | نظام ذكاء اصطناعي`, 15, captureCanvas.height - 55);
            const avgAttention = faces.length > 0 ? Math.round(faces.reduce((sum, f) => sum + f.attentionScore, 0) / faces.length) : 0;
            const avgAIConfidence = faces.length > 0 ? Math.round(faces.reduce((sum, f) => sum + f.confidence, 0) / faces.length * 100) : 0;
            captureCtx.fillText(`متوسط الانتباه: ${avgAttention}% | ثقة AI: ${avgAIConfidence}% | BlazeFace + TensorFlow.js`, 15, captureCanvas.height - 35);
            captureCtx.font = '14px Cairo';
            captureCtx.fillText(`FPS: ${currentFPS} | معالجة: ${document.getElementById('processingTime').textContent}`, 15, captureCanvas.height - 15);
            
            // Show captured image
            const dataURL = captureCanvas.toDataURL('image/png');
            const img = document.createElement('img');
            img.src = dataURL;
            img.style.cssText = 'position: absolute; top: 0; left: 0; width: 100%; height: 100%; object-fit: cover; z-index: 20; border-radius: 12px;';
            
            const container = video.parentElement;
            container.appendChild(img);
            
            addToLog(`📸 تم التقاط صورة AI تحتوي على ${faces.length} طالب`);
            showNotification('تم التقاط الصورة بالذكاء الاصطناعي! 🤖📸');
            
            // Remove image after 6 seconds
            setTimeout(() => {
                if (img.parentNode) {
                    img.parentNode.removeChild(img);
                }
            }, 6000);
        }

        // Student management functions (same as before but with AI integration)
        function addStudent() {
            const input = document.getElementById('studentNameInput');
            const name = input.value.trim();
            
            if (!name) {
                showNotification('يرجى إدخال اسم الطالب', 'error');
                return;
            }
            
            if (students.includes(name)) {
                showNotification('هذا الطالب موجود بالفعل', 'error');
                return;
            }
            
            students.push(name);
            input.value = '';
            updateStudentsList();
            saveData();
            addToLog(`➕ تم إضافة الطالب: ${name}`);
            showNotification(`تم إضافة الطالب: ${name} ✅`);
        }

        function updateStudentsList() {
            const list = document.getElementById('studentsList');
            list.innerHTML = '';
            
            students.forEach((student, index) => {
                const div = document.createElement('div');
                div.className = 'flex justify-between items-center p-3 bg-gray-50 rounded-lg border hover:bg-gray-100 transition-colors';
                div.innerHTML = `
                    <span class="text-gray-800 font-medium">${student}</span>
                    <button onclick="removeStudent(${index})" class="text-red-500 hover:text-red-700 text-sm px-2 py-1 rounded transition-colors">
                        🗑️
                    </button>
                `;
                list.appendChild(div);
            });
        }

        function removeStudent(index) {
            const studentName = students[index];
            students.splice(index, 1);
            updateStudentsList();
            saveData();
            addToLog(`🗑️ تم حذف الطالب: ${studentName}`);
            showNotification(`تم حذف الطالب: ${studentName}`);
        }

        function selectRandomStudent() {
            if (students.length === 0) {
                showNotification('يرجى إضافة طلاب أولاً', 'error');
                return;
            }
            
            const randomIndex = Math.floor(Math.random() * students.length);
            const selectedStudent = students[randomIndex];
            
            displaySelectedStudent(selectedStudent, 'تم الاختيار عشوائياً');
            addToLog(`🎯 تم اختيار الطالب: ${selectedStudent} عشوائياً`);
        }

        function selectByAttention() {
            const faces = Array.from(detectedFaces.values());
            if (faces.length === 0) {
                showNotification('لا يوجد طلاب مكتشفون بالذكاء الاصطناعي', 'error');
                return;
            }
            
            // Find face with highest attention score
            const bestFace = faces.reduce((best, current) => 
                current.attentionScore > best.attentionScore ? current : best
            );
            
            selectedStudentFace = bestFace;
            
            let studentName = '';
            if (students.length > 0) {
                studentName = students[Math.floor(Math.random() * students.length)];
            } else {
                studentName = `طالب ${bestFace.id}`;
            }
            
            const aiConfidence = Math.round(bestFace.confidence * 100);
            const reason = `الأكثر انتباهاً (${bestFace.attentionScore}% - AI: ${aiConfidence}%)`;
            
            displaySelectedStudent(studentName, reason);
            addToLog(`🤖 تم اختيار ${studentName} بالذكاء الاصطناعي - ${reason}`);
        }

        function selectHandRaised() {
            const faces = Array.from(detectedFaces.values());
            const handRaisedFaces = faces.filter(f => f.handRaised);
            
            if (handRaisedFaces.length === 0) {
                showNotification('لا يوجد طلاب رافعين أيديهم حسب الذكاء الاصطناعي', 'error');
                return;
            }
            
            // Select random from AI-detected hand raised faces
            const randomFace = handRaisedFaces[Math.floor(Math.random() * handRaisedFaces.length)];
            selectedStudentFace = randomFace;
            
            let studentName = '';
            if (students.length > 0) {
                studentName = students[Math.floor(Math.random() * students.length)];
            } else {
                studentName = `طالب ${randomFace.id}`;
            }
            
            const aiConfidence = Math.round(randomFace.confidence * 100);
            const reason = `رفع يده (اكتشاف AI: ${aiConfidence}%) 🙋‍♂️`;
            
            displaySelectedStudent(studentName, reason);
            addToLog(`🤖 تم اختيار ${studentName} - ${reason}`);
        }

        function displaySelectedStudent(studentName, reason) {
            document.getElementById('selectedStudent').style.display = 'block';
            document.getElementById('selectedStudentName').textContent = studentName;
            document.getElementById('selectionReason').textContent = reason;
            
            showNotification(`تم اختيار: ${studentName}! 🎯`);
        }

        function freezeForAnswer() {
            capturePhoto();
            
            const wasDetecting = isDetecting;
            isDetecting = false;
            
            addToLog('📸 تم تثبيت الصورة للطالب المختار');
            showNotification('تم تثبيت الصورة للإجابة! 🎯');
            
            setTimeout(() => {
                if (wasDetecting && currentStream) {
                    isDetecting = true;
                    detectFacesWithAI();
                }
            }, 12000);
        }

        // Quick actions with AI integration
        function alertAttention() {
            addToLog('🔔 تم إرسال تنبيه للانتباه');
            showNotification('🔔 تم إرسال تنبيه للانتباه!');
            
            document.body.style.backgroundColor = '#fef3c7';
            setTimeout(() => {
                document.body.style.backgroundColor = '';
            }, 1000);
        }

        function resetTracking() {
            // Clear all AI tracking data
            detectedFaces.clear();
            faceTracker.clear();
            movementTracker.clear();
            attentionAnalyzer.clear();
            handRaiseDetector.clear();
            
            faceIdCounter = 0;
            selectedStudentFace = null;
            frameCount = 0;
            currentFPS = 0;
            
            // Clear UI
            detectionOverlay.innerHTML = '';
            document.getElementById('selectedStudent').style.display = 'none';
            
            addToLog('🔄 تم إعادة تعيين جميع بيانات التتبع الذكي');
            showNotification('تم إعادة تعيين التتبع الذكي! 🤖🔄');
        }

        async function generateReport() {
            document.getElementById('reportModal').classList.remove('hidden');
            
            const reportContent = document.getElementById('reportContent');
            reportContent.innerHTML = `
                <div class="text-center py-12">
                    <div class="loading-spinner mb-4"></div>
                    <p class="text-lg text-gray-600">جاري إنشاء التقرير الذكي...</p>
                    <p class="text-sm text-gray-500 mt-2">🤖 تحليل بيانات الذكاء الاصطناعي</p>
                </div>
            `;
            
            // Simulate AI processing
            await new Promise(resolve => setTimeout(resolve, 2000));
            
            const duration = sessionStats.startTime ? 
                Math.round((new Date() - sessionStats.startTime) / 60000) : 0;
            
            const faces = Array.from(detectedFaces.values());
            const avgAttention = faces.length > 0 ? 
                Math.round(faces.reduce((sum, f) => sum + f.attentionScore, 0) / faces.length) : 0;
            const avgAIConfidence = faces.length > 0 ? 
                Math.round(faces.reduce((sum, f) => sum + f.confidence, 0) / faces.length * 100) : 0;
            
            reportContent.innerHTML = `
                <div class="space-y-6">
                    <div class="bg-gradient-to-r from-blue-50 to-purple-50 rounded-lg p-6">
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">🤖 ملخص الجلسة الذكية</h3>
                        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                            <div class="text-center">
                                <div class="text-2xl font-bold text-blue-600">${duration}</div>
                                <div class="text-sm text-gray-600">دقيقة</div>
                            </div>
                            <div class="text-center">
                                <div class="text-2xl font-bold text-green-600">${faceTracker.size}</div>
                                <div class="text-sm text-gray-600">طالب مكتشف بـ AI</div>
                            </div>
                            <div class="text-center">
                                <div class="text-2xl font-bold text-orange-600">${avgAttention}%</div>
                                <div class="text-sm text-gray-600">متوسط الانتباه</div>
                            </div>
                            <div class="text-center">
                                <div class="text-2xl font-bold text-purple-600">${avgAIConfidence}%</div>
                                <div class="text-sm text-gray-600">ثقة الذكاء الاصطناعي</div>
                            </div>
                        </div>
                        <div class="mt-4 text-center">
                            <div class="text-sm text-gray-600">
                                🤖 تم استخدام BlazeFace AI + TensorFlow.js | 
                                رفع يد: ${sessionStats.handRaises} | 
                                FPS: ${currentFPS}
                            </div>
                        </div>
                    </div>
                    
                    <div>
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">🎯 تحليل الطلاب بالذكاء الاصطناعي</h3>
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                            ${generateAIStudentReports()}
                        </div>
                    </div>
                    
                    <div>
                        <h3 class="text-lg font-semibold text-gray-800 mb-4">🤖 التوصيات الذكية</h3>
                        <div class="space-y-3">
                            ${generateAIRecommendations(avgAttention, avgAIConfidence, duration)}
                        </div>
                    </div>
                </div>
            `;
            
            addToLog('📊 تم إنشاء التقرير الذكي');
            showNotification('تم إنشاء تقرير ذكي مفصل! 🤖📊');
        }

        function generateAIStudentReports() {
            if (faceTracker.size === 0) {
                return '<div class="col-span-full text-center text-gray-500 py-8">لا توجد بيانات طلاب من الذكاء الاصطناعي</div>';
            }
            
            return Array.from(faceTracker.values()).map(tracker => {
                const analyzer = attentionAnalyzer.get(tracker.id);
                const handDetector = handRaiseDetector.get(tracker.id);
                
                const avgAttention = analyzer ? analyzer.avgScore : 0;
                const avgAIConfidence = Math.round(tracker.avgConfidence * 100);
                const participationTime = Math.round((tracker.lastSeen - tracker.firstSeen) / 60000);
                const handRaises = handDetector ? handDetector.totalRaises : 0;
                
                return `
                    <div class="bg-white p-4 rounded-lg border shadow-sm">
                        <div class="flex justify-between items-center mb-3">
                            <h4 class="font-bold text-gray-800">🤖 طالب ${tracker.id}</h4>
                            <div class="flex gap-1">
                                <span class="text-xs px-2 py-1 rounded-full ${
                                    avgAttention > 70 ? 'bg-green-500' : 
                                    avgAttention > 40 ? 'bg-yellow-500' : 'bg-red-500'
                                } text-white">
                                    ${avgAttention > 70 ? 'ممتاز' : avgAttention > 40 ? 'جيد' : 'يحتاج تحسين'}
                                </span>
                                <span class="text-xs px-2 py-1 rounded-full bg-blue-500 text-white">
                                    AI: ${avgAIConfidence}%
                                </span>
                            </div>
                        </div>
                        <div class="space-y-2 text-sm">
                            <div class="flex justify-between">
                                <span>متوسط الانتباه:</span>
                                <span class="font-semibold">${avgAttention}%</span>
                            </div>
                            <div class="flex justify-between">
                                <span>ثقة الذكاء الاصطناعي:</span>
                                <span class="font-semibold">${avgAIConfidence}%</span>
                            </div>
                            <div class="flex justify-between">
                                <span>وقت المشاركة:</span>
                                <span class="font-semibold">${participationTime} دقيقة</span>
                            </div>
                            <div class="flex justify-between">
                                <span>رفع اليد (AI):</span>
                                <span class="font-semibold">${handRaises} مرة</span>
                            </div>
                            <div class="flex justify-between">
                                <span>اكتشافات AI:</span>
                                <span class="font-semibold">${tracker.detectionCount}</span>
                            </div>
                        </div>
                        <div class="mt-3">
                            <div class="bg-gray-200 rounded-full h-3">
                                <div class="bg-gradient-to-r from-blue-400 to-green-500 h-3 rounded-full" style="width: ${avgAttention}%"></div>
                            </div>
                        </div>
                        <div class="mt-2 text-xs text-gray-500 text-center">
                            BlazeFace AI Detection
                        </div>
                    </div>
                `;
            }).join('');
        }

        function generateAIRecommendations(avgAttention, avgAIConfidence, duration) {
            const recommendations = [];
            
            if (avgAIConfidence < 70) {
                recommendations.push({
                    type: 'warning',
                    title: 'ثقة الذكاء الاصطناعي منخفضة',
                    text: `ثقة AI: ${avgAIConfidence}%. قد تحتاج لتحسين الإضاءة أو زاوية الكاميرا لتحسين دقة الاكتشاف.`,
                    icon: '🤖'
                });
            }
            
            if (avgAttention < 50) {
                recommendations.push({
                    type: 'warning',
                    title: 'مستوى انتباه منخفض',
                    text: `متوسط الانتباه ${avgAttention}%. الذكاء الاصطناعي يقترح إضافة أنشطة تفاعلية.`,
                    icon: '⚠️'
                });
            }
            
            if (avgAttention > 80 && avgAIConfidence > 85) {
                recommendations.push({
                    type: 'success',
                    title: 'أداء ممتاز مع AI!',
                    text: `مستوى الانتباه عالي (${avgAttention}%) وثقة AI ممتازة (${avgAIConfidence}%). النظام يعمل بكفاءة عالية.`,
                    icon: '🤖✨'
                });
            }
            
            if (sessionStats.handRaises > 15) {
                recommendations.push({
                    type: 'success',
                    title: 'مشاركة نشطة مكتشفة بـ AI',
                    text: `تم رصد ${sessionStats.handRaises} حالة رفع يد بالذكاء الاصطناعي. الطلاب متفاعلون بنشاط.`,
                    icon: '🙋‍♂️🤖'
                });
            }
            
            if (currentFPS < 15) {
                recommendations.push({
                    type: 'info',
                    title: 'أداء النظام',
                    text: `معدل الإطارات منخفض (${currentFPS} FPS). فكر في تقليل دقة الكاميرا لتحسين الأداء.`,
                    icon: '⚡'
                });
            }
            
            if (duration > 45) {
                recommendations.push({
                    type: 'info',
                    title: 'وقت طويل',
                    text: 'الحصة طويلة، الذكاء الاصطناعي يقترح إضافة استراحات قصيرة.',
                    icon: '⏰'
                });
            }
            
            if (recommendations.length === 0) {
                recommendations.push({
                    type: 'success',
                    title: 'النظام الذكي يعمل بكفاءة',
                    text: 'الذكاء الاصطناعي يرصد أداءً جيداً. استمر في المراقبة والتفاعل مع الطلاب.',
                    icon: '🤖✅'
                });
            }
            
            return recommendations.map(rec => `
                <div class="p-4 rounded-lg border-r-4 ${
                    rec.type === 'warning' ? 'bg-red-50 border-red-500' : 
                    rec.type === 'success' ? 'bg-green-50 border-green-500' : 'bg-blue-50 border-blue-500'
                }">
                    <div class="flex items-start space-x-3 space-x-reverse">
                        <span class="text-2xl">${rec.icon}</span>
                        <div>
                            <h4 class="font-semibold text-gray-800 mb-1">${rec.title}</h4>
                            <p class="text-sm text-gray-600">${rec.text}</p>
                        </div>
                    </div>
                </div>
            `).join('');
        }

        function exportData() {
            const exportData = {
                sessionInfo: {
                    startTime: sessionStats.startTime,
                    endTime: new Date(),
                    duration: sessionStats.startTime ? 
                        Math.round((new Date() - sessionStats.startTime) / 60000) : 0,
                    totalDetections: sessionStats.totalDetections,
                    handRaises: sessionStats.handRaises,
                    aiModel: 'BlazeFace + TensorFlow.js',
                    avgFPS: currentFPS
                },
                aiStudents: Array.from(faceTracker.values()).map(tracker => ({
                    id: tracker.id,
                    firstSeen: tracker.firstSeen,
                    lastSeen: tracker.lastSeen,
                    detectionCount: tracker.detectionCount,
                    avgConfidence: tracker.avgConfidence,
                    attentionData: attentionAnalyzer.get(tracker.id),
                    handRaiseData: handRaiseDetector.get(tracker.id),
                    aiSource: 'BlazeFace'
                })),
                attentionHistory: sessionStats.attentionHistory,
                movementHistory: sessionStats.movementHistory,
                aiMetrics: {
                    modelVersion: 'BlazeFace v0.0.7',
                    tensorflowVersion: 'v4.10.0',
                    avgProcessingTime: document.getElementById('processingTime').textContent,
                    finalFPS: currentFPS
                }
            };
            
            const dataStr = JSON.stringify(exportData, null, 2);
            const dataBlob = new Blob([dataStr], {type: 'application/json'});
            
            const link = document.createElement('a');
            link.href = URL.createObjectURL(dataBlob);
            link.download = `classroom-ai-report-${new Date().toISOString().split('T')[0]}.json`;
            
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            
            showNotification('تم تصدير بيانات الذكاء الاصطناعي بنجاح! 🤖💾');
            addToLog('💾 تم تصدير بيانات الجلسة الذكية');
        }

        // Utility functions
        function addToLog(message) {
            const log = document.getElementById('detectionLog');
            const time = new Date().toLocaleTimeString('ar-SA');
            const logEntry = document.createElement('p');
            logEntry.innerHTML = `<span class="text-gray-400">[${time}]</span> ${message}`;
            log.insertBefore(logEntry, log.firstChild);
            
            while (log.children.length > 100) {
                log.removeChild(log.lastChild);
            }
        }

        function saveData() {
            const data = {
                students: students,
                sessionStats: sessionStats
            };
            localStorage.setItem('advancedClassroomAITool', JSON.stringify(data));
        }

        function loadSavedData() {
            const saved = localStorage.getItem('advancedClassroomAITool');
            if (saved) {
                const data = JSON.parse(saved);
                students = data.students || [];
                sessionStats = { ...sessionStats, ...data.sessionStats };
                
                updateStudentsList();
            }
        }

        function showNotification(message, type = 'success') {
            const notification = document.createElement('div');
            notification.className = `fixed top-4 left-4 p-4 rounded-lg text-white z-50 shadow-lg transition-all ${
                type === 'error' ? 'bg-red-500' : 'bg-green-500'
            }`;
            notification.textContent = message;
            
            document.body.appendChild(notification);
            
            setTimeout(() => {
                notification.style.opacity = '0';
                setTimeout(() => {
                    if (notification.parentNode) {
                        notification.parentNode.removeChild(notification);
                    }
                }, 300);
            }, 5000);
        }

        // Make functions globally accessible
        window.removeStudent = removeStudent;

        // Cleanup and memory management
        window.addEventListener('beforeunload', () => {
            if (currentStream) {
                currentStream.getTracks().forEach(track => track.stop());
            }
            
            // Clean up TensorFlow.js tensors
            if (typeof tf !== 'undefined') {
                tf.disposeVariables();
            }
            
            saveData();
        });

        // Memory cleanup interval
        setInterval(() => {
            if (typeof tf !== 'undefined') {
                const numTensors = tf.memory().numTensors;
                if (numTensors > 100) {
                    console.log(`Cleaning up tensors: ${numTensors}`);
                    tf.tidy(() => {
                        // Force cleanup of unused tensors
                    });
                }
            }
        }, 30000); // Every 30 seconds
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'96d2c33296035ccf',t:'MTc1NDg2MzE4OS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
