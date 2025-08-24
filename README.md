# Watermark_Removal
Remove watermark and unwanted text in photo

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image Enhancer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }

        .container {
            max-width: 900px;
            margin: auto;
            padding: 2rem;
        }

        .card {
            background-color: #ffffff;
            border-radius: 1rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            padding: 2rem;
        }

        .input-group label {
            font-weight: 600;
            color: #4b5563;
        }

        .input-group input,
        .input-group textarea {
            border: 1px solid #d1d5db;
            border-radius: 0.5rem;
            padding: 0.75rem;
            transition: border-color 0.2s;
            background-color: #f9fafb;
        }

        .input-group input:focus,
        .input-group textarea:focus {
            outline: none;
            border-color: #4f46e5;
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
        }

        .btn {
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            font-weight: 600;
            color: #ffffff;
            transition: background-color 0.2s, transform 0.1s;
        }

        .btn-primary {
            background-color: #4f46e5;
        }

        .btn-primary:hover {
            background-color: #4338ca;
            transform: translateY(-1px);
        }

        .spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top-color: white;
            border-radius: 50%;
            width: 1.5rem;
            height: 1.5rem;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            to {
                transform: rotate(360deg);
            }
        }

        .fade-in {
            animation: fadeIn 0.5s ease-out;
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(10px);
            }

            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
    </style>
</head>

<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div class="container">
        <div class="card space-y-6">
            <div class="text-center">
                <h1 class="text-3xl font-bold text-gray-800">Image Enhancer</h1>
                <p class="text-gray-500 mt-2">Upload an image and describe how you want to enhance it.</p>
            </div>

            <div class="space-y-4">
                <div class="input-group">
                    <label for="imageUpload" class="block mb-2 text-sm">Upload Image</label>
                    <input type="file" id="imageUpload" accept="image/*"
                        class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-violet-50 file:text-violet-700 hover:file:bg-violet-100 cursor-pointer">
                </div>

                <div class="input-group">
                    <label for="prompt" class="block mb-2 text-sm">Prompt</label>
                    <textarea id="prompt" rows="3"
                        class="w-full transition-all duration-300 resize-none focus:h-24 h-12"
                        placeholder="e.g., 'a high-resolution portrait, no watermark, cinematic lighting'"></textarea>
                </div>
            </div>

            <div class="flex items-center justify-between">
                <button id="generateBtn" class="btn btn-primary flex items-center justify-center space-x-2 w-full"
                    disabled>
                    <span id="btnText">Generate Image</span>
                    <div id="btnSpinner" class="hidden spinner"></div>
                </button>
            </div>

            <div id="outputContainer" class="mt-8 hidden fade-in">
                <h2 class="text-xl font-semibold text-gray-800 mb-4 text-center">Result</h2>
                <div class="flex justify-center items-center h-full">
                    <img id="outputImage" src="#" alt="Generated Image" class="w-full h-auto rounded-lg shadow-md max-w-sm">
                </div>
                <p id="errorMessage" class="text-red-500 text-center mt-4 hidden">Error generating image. Please try again.</p>
            </div>

            <div id="messageBox" class="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center hidden">
                <div class="bg-white rounded-lg p-6 shadow-xl max-w-sm mx-auto">
                    <h3 class="text-lg font-bold text-gray-800">Notice</h3>
                    <p class="mt-2 text-sm text-gray-600" id="messageText"></p>
                    <div class="flex justify-end mt-4">
                        <button onclick="document.getElementById('messageBox').classList.add('hidden')"
                            class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700">OK</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Use a random ID for this instance to ensure proper data isolation
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        const imageUpload = document.getElementById('imageUpload');
        const promptInput = document.getElementById('prompt');
        const generateBtn = document.getElementById('generateBtn');
        const btnText = document.getElementById('btnText');
        const btnSpinner = document.getElementById('btnSpinner');
        const outputContainer = document.getElementById('outputContainer');
        const outputImage = document.getElementById('outputImage');
        const errorMessage = document.getElementById('errorMessage');
        const messageBox = document.getElementById('messageBox');
        const messageText = document.getElementById('messageText');

        let uploadedBase64 = null;

        function showMessage(text) {
            messageText.textContent = text;
            messageBox.classList.remove('hidden');
        }

        imageUpload.addEventListener('change', function (e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onloadend = () => {
                    uploadedBase64 = reader.result.split(',')[1];
                    generateBtn.disabled = false;
                };
                reader.readAsDataURL(file);
            }
        });

        generateBtn.addEventListener('click', async () => {
            if (!uploadedBase64) {
                showMessage('Please upload an image first.');
                return;
            }

            // Show loading state
            btnText.textContent = 'Generating...';
            btnSpinner.classList.remove('hidden');
            generateBtn.disabled = true;
            outputContainer.classList.add('hidden');
            errorMessage.classList.add('hidden');

            const userPrompt = promptInput.value || "a high-resolution photo, professional photography style, no watermark, enhanced details";
            const payload = {
                contents: [{
                    parts: [{
                        inlineData: {
                            mimeType: "image/png",
                            data: uploadedBase64
                        }
                    }, {
                        text: userPrompt
                    }]
                }],
                generationConfig: {
                    responseModalities: ["IMAGE"]
                }
            };

            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-preview-image-generation:generateContent?key=${apiKey}`;

            try {
                let response = null;
                let retries = 0;
                const maxRetries = 3;
                while (retries < maxRetries) {
                    try {
                        response = await fetch(apiUrl, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });
                        if (response.ok) {
                            break;
                        } else if (response.status === 429) {
                            retries++;
                            const delay = Math.pow(2, retries) * 1000;
                            await new Promise(resolve => setTimeout(resolve, delay));
                        } else {
                            throw new Error(`HTTP error! status: ${response.status}`);
                        }
                    } catch (error) {
                        if (retries === maxRetries - 1) {
                            throw error;
                        }
                        retries++;
                        const delay = Math.pow(2, retries) * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                    }
                }
                const result = await response.json();
                
                const base64Data = result?.candidates?.[0]?.content?.parts?.find(p => p.inlineData)?.inlineData?.data;

                if (base64Data) {
                    const imageUrl = `data:image/png;base64,${base64Data}`;
                    outputImage.src = imageUrl;
                    outputContainer.classList.remove('hidden');
                } else {
                    errorMessage.classList.remove('hidden');
                }
            } catch (error) {
                console.error("Error generating image:", error);
                errorMessage.classList.remove('hidden');
            } finally {
                btnText.textContent = 'Generate Image';
                btnSpinner.classList.add('hidden');
                generateBtn.disabled = false;
            }
        });
    </script>
</body>

</html>
