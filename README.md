<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AID 처분 방법 안내</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Desert Neutrals -->
    <!-- Application Structure Plan: 사용자가 자신의 '화이트리스트/KYC 상태'라는 단일 질문에 버튼으로 답하면, 그에 해당하는 '상환' 또는 '교환' 방법만 표시되는 대화형 의사결정 트리 모델을 사용했습니다. 이 작업 중심의 흐름은 보고서의 정보를 단순히 나열하는 것보다 사용자가 불필요한 정보를 건너뛰고 자신에게 맞는 경로를 쉽게 찾도록 하여 사용성을 높입니다. Firebase를 연동하여 선택 상태를 저장하고, 고유 ID가 포함된 URL을 통해 다른 사용자와 선택 결과를 공유할 수 있도록 구조를 확장했습니다. -->
    <!-- Visualization & Content Choices: 
        - 보고서 정보: KYC/화이트리스트 상태에 따른 사용자 구분 -> 목표: 사용자 안내 -> 표현 방식: 대화형 버튼 -> 상호작용: 클릭 시 정보 표시 및 Firestore에 상태 저장 -> 정당성: 사용자 여정 단순화 및 공유 기능 구현.
        - 보고서 정보: 상환(Redeeming) 절차 -> 목표: 정보 제공 -> 표현 방식: HTML/CSS 순서도 + 텍스트 + 아이콘 -> 상호작용: 클릭 시 표시 -> 정당성: 직접적인 1:1 교환 과정을 시각적으로 명료하게 전달. 구현: HTML/Tailwind.
        - 보고서 정보: 교환(Swapping) 절차 -> 목표: 정보 제공 -> 표현 방식: HTML/CSS 순서도 + 텍스트 + 아이콘 -> 상호작용: 클릭 시 표시 -> 정당성: 외부 풀을 통한 간접 교환 과정을 시각적으로 명료하게 전달. 구현: HTML/Tailwind.
        - Gemini Feature: AI 상세 설명 -> 목표: 심층 정보 제공 -> 표현 방식: 버튼 클릭 후 동적 텍스트 표시 -> 상호작용: 클릭 시 Gemini API 호출로 설명 생성 -> 정당성: 사용자의 추가적인 궁금증을 해소하고 복잡한 개념을 쉽게 풀어줌.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
        }
        .flow-diagram {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 0.5rem;
            flex-wrap: wrap;
        }
        .flow-item {
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            text-align: center;
            font-weight: 500;
        }
        .flow-arrow {
            font-size: 1.5rem;
            font-weight: bold;
            color: #9ca3af;
        }
        .btn-option {
            transition: all 0.3s ease;
            border-width: 2px;
        }
        .btn-option.selected {
            transform: translateY(-4px);
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
        }
        .btn-option:disabled {
            cursor: not-allowed;
            opacity: 0.7;
        }
        .ai-explanation {
            white-space: pre-wrap;
            word-wrap: break-word;
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #D35400;
            border-radius: 50%;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-[#FDF6E3] text-[#584128]">
    <div class="container mx-auto max-w-4xl p-4 sm:p-6 lg:p-8 min-h-screen flex flex-col items-center justify-center">
        
        <div id="loader" class="text-center">
            <p class="text-lg">페이지를 불러오는 중입니다...</p>
        </div>
        
        <div id="app-content" class="hidden w-full">
            <header class="text-center mb-8">
                <div class="flex justify-center mb-4">
                     <img src="https://i.imgur.com/MuJymoj.png" alt="AID 캐릭터" class="w-auto h-48 rounded-lg shadow-md"
                     onerror="this.onerror=null; this.src='https://placehold.co/400x192/FDF6E3/584128?text=GAIB';">
                </div>
                <h1 class="text-4xl font-bold text-[#D35400] mb-2">AID 처분 방법</h1>
                <p class="text-lg text-[#7E6245]">당신의 상황에 맞는 가장 효율적인 방법을 찾아보세요.</p>
            </header>
    
            <main class="w-full bg-white/50 backdrop-blur-sm p-6 sm:p-8 rounded-2xl shadow-lg border border-black/10">
                <div id="error-message" class="hidden text-center font-bold p-4 rounded-lg mb-4"></div>
                <section id="decision-section" class="text-center">
                    <h2 class="text-2xl font-bold mb-6">화이트리스트에 등록되고 KYC를 완료하셨나요?</h2>
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <button id="btn-yes" class="btn-option w-full p-6 bg-[#2ECC71]/80 hover:bg-[#2ECC71] text-white rounded-lg border-2 border-transparent hover:border-white">
                            <span class="text-xl font-bold">✅ 예, 완료했습니다</span>
                        </button>
                        <button id="btn-no" class="btn-option w-full p-6 bg-[#E74C3C]/80 hover:bg-[#E74C3C] text-white rounded-lg border-2 border-transparent hover:border-white">
                            <span class="text-xl font-bold">❌ 아니오, 해당되지 않습니다</span>
                        </button>
                    </div>
                </section>
    
                <section id="result-section" class="mt-8 transition-opacity duration-500 ease-in-out">
                    <div id="redeem-info" class="hidden p-6 bg-green-50 rounded-lg border-2 border-green-200">
                        <div class="flex items-center mb-4">
                            <img src="https://i.imgur.com/3Z2i4sW.png" alt="AID 캐릭터 아이콘" class="w-12 h-12 rounded-full mr-4 border-2 border-green-200" onerror="this.onerror=null; this.src='https://placehold.co/48x48/e0f2f1/105d46?text=:)';">
                            <h3 class="text-2xl font-bold text-green-800">상환 (Redeeming)</h3>
                        </div>
                        <p class="text-green-700 mb-6">화이트리스트에 등록되고 KYC를 완료한 사용자는 GAIB 프로토콜을 통해 AID를 직접 상환할 수 있습니다. AID를 예치하면, 1:1 비율로 스테이블코인을 받을 수 있습니다.</p>
                        <div class="flow-diagram mb-6">
                            <div class="flow-item bg-green-200 text-green-800">사용자</div>
                            <div class="flow-arrow text-green-500">→</div>
                            <div class="flow-item bg-green-200 text-green-800">GAIB 프로토콜</div>
                            <div class="flow-arrow text-green-500">→</div>
                            <div class="flow-item bg-green-200 text-green-800">1:1 스테이블코인</div>
                        </div>
                        <button class="ai-explain-btn w-full bg-green-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-700 transition" data-method="redeem">✨ AI로 더 자세히 알아보기</button>
                        <div id="redeem-ai-explanation-container" class="mt-4 hidden">
                            <h4 class="font-bold text-lg text-green-800 mb-2">✨ AI가 생성한 상세 설명</h4>
                            <div class="loader-container flex justify-center p-4"><div class="loader"></div></div>
                            <p class="ai-explanation bg-green-100/50 p-4 rounded-md text-green-900"></p>
                        </div>
                    </div>
    
                    <div id="swap-info" class="hidden p-6 bg-red-50 rounded-lg border-2 border-red-200">
                         <div class="flex items-center mb-4">
                            <img src="https://i.imgur.com/3Z2i4sW.png" alt="AID 캐릭터 아이콘" class="w-12 h-12 rounded-full mr-4 border-2 border-red-200" onerror="this.onerror=null; this.src='https://placehold.co/48x48/ffebee/b91c1c?text=:)';">
                            <h3 class="text-2xl font-bold text-red-800">교환 (Swapping)</h3>
                        </div>
                        <p class="text-red-700 mb-6">화이트리스트에 등록되지 않은 사용자는 GAIB 사용자 인터페이스를 이용해 외부 유동성 풀에 접근함으로써 AID를 스테이블코인으로 교환할 수 있습니다.</p>
                         <div class="flow-diagram mb-6">
                            <div class="flow-item bg-red-200 text-red-800">사용자</div>
                            <div class="flow-arrow text-red-500">→</div>
                            <div class="flow-item bg-red-200 text-red-800">GAIB UI</div>
                            <div class="flow-arrow text-red-500">→</div>
                            <div class="flow-item bg-red-200 text-red-800">외부 유동성 풀</div>
                            <div class="flow-arrow text-red-500">→</div>
                            <div class="flow-item bg-red-200 text-red-800">스테이블코인</div>
                        </div>
                        <button class="ai-explain-btn w-full bg-red-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-red-700 transition" data-method="swap">✨ AI로 더 자세히 알아보기</button>
                        <div id="swap-ai-explanation-container" class="mt-4 hidden">
                            <h4 class="font-bold text-lg text-red-800 mb-2">✨ AI가 생성한 상세 설명</h4>
                            <div class="loader-container flex justify-center p-4"><div class="loader"></div></div>
                            <p class="ai-explanation bg-red-100/50 p-4 rounded-md text-red-900"></p>
                        </div>
                    </div>
                </section>
            </main>
            
            <footer class="text-center mt-8 text-sm text-[#7E6245] w-full">
                <div class="flex flex-col items-center gap-4">
                    <button id="share-btn" class="bg-[#D35400] text-white font-bold py-2 px-6 rounded-full hover:bg-[#A04000] transition-transform transform hover:scale-105 disabled:bg-gray-400 disabled:cursor-not-allowed" disabled>
                        이 페이지 공유하기 🔗
                    </button>
                    <p class="mt-2">&copy; GAIB Protocol Guide. All rights reserved.</p>
                </div>
            </footer>
        </div>
    </div>
    
    <div id="toast" class="fixed bottom-10 left-1/2 -translate-x-1/2 bg-gray-900 text-white py-2 px-5 rounded-full shadow-lg opacity-0 transition-opacity duration-500 z-50">
        링크가 클립보드에 복사되었습니다!
    </div>

    <script type="module">
        const loader = document.getElementById('loader');
        const appContent = document.getElementById('app-content');
        const errorDiv = document.getElementById('error-message');
        const btnYes = document.getElementById('btn-yes');
        const btnNo = document.getElementById('btn-no');
        const shareBtn = document.getElementById('share-btn');
        const redeemInfo = document.getElementById('redeem-info');
        const swapInfo = document.getElementById('swap-info');
        const toast = document.getElementById('toast');

        // --- ✏️ 사용자 설정 영역 ---
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_PROJECT_ID.appspot.com",
            messagingSenderId: "YOUR_SENDER_ID",
            appId: "YOUR_APP_ID"
        };
        // --- ✏️ 사용자 설정 영역 끝 ---

        const isFirebaseConfigured = firebaseConfig.apiKey && firebaseConfig.apiKey !== "YOUR_API_KEY";

        const showInfo = (type) => {
            if (type === 'redeem') {
                redeemInfo.classList.remove('hidden');
                swapInfo.classList.add('hidden');
                btnYes.classList.add('selected', 'bg-[#2ECC71]');
                btnNo.classList.remove('selected', 'bg-[#E74C3C]');
                btnNo.classList.add('bg-[#E74C3C]/80');
            } else if (type === 'swap') {
                redeemInfo.classList.add('hidden');
                swapInfo.classList.remove('hidden');
                btnNo.classList.add('selected', 'bg-[#E74C3C]');
                btnYes.classList.remove('selected', 'bg-[#2ECC71]');
                btnYes.classList.add('bg-[#2ECC71]/80');
            }
             if (isFirebaseConfigured) {
                shareBtn.disabled = false;
            }
        };

        const handleChoice = (choice) => {
            showInfo(choice);
        };

        btnYes.addEventListener('click', () => handleChoice('redeem'));
        btnNo.addEventListener('click', () => handleChoice('swap'));
        
        loader.classList.add('hidden');
        appContent.classList.remove('hidden');
        
        // --- Gemini AI Feature ---
        const getAiExplanation = async (method) => {
            const container = document.getElementById(`${method}-ai-explanation-container`);
            const textElement = container.querySelector('.ai-explanation');
            const loaderElement = container.querySelector('.loader-container');
            const button = document.querySelector(`.ai-explain-btn[data-method="${method}"]`);

            container.classList.remove('hidden');
            textElement.classList.add('hidden');
            loaderElement.classList.remove('hidden');
            button.disabled = true;

            let prompt = "";
            if (method === 'redeem') {
                prompt = "당신은 암호화폐 개념을 쉽게 설명하는 전문가입니다. 사용자는 'GAIB 프로토콜'에서 'AID'라는 디지털 자산을 '상환(Redeeming)'하는 방법에 대해 궁금해합니다. 이 사용자는 화이트리스트에 등록되었고 KYC를 완료했습니다. 상환 과정은 AID를 예치하면 스테이블코인을 1:1 비율로 받는 것입니다. 이 과정을 단계별로 설명하고, '스테이블코인'이 무엇인지, 그리고 이 방법이 왜 프로토콜을 통한 직접적이고 안전한 방법인지 간단히 설명해주세요.";
            } else { // swap
                prompt = "당신은 암호화폐 개념을 쉽게 설명하는 전문가입니다. 사용자는 'GAIB 프로토콜'에서 'AID'라는 디지털 자산을 '교환(Swapping)'하는 방법에 대해 궁금해합니다. 이 사용자는 화이트리스트에 등록되지 않았습니다. 교환 과정은 GAIB 사용자 인터페이스를 통해 외부 유동성 풀에 접근하여 AID를 스테이블코인으로 바꾸는 것입니다. 이 과정을 단계별로 설명하고, '외부 유동성 풀'이 무엇인지, 그리고 이 방법이 '상환'과 어떻게 다른지 간단히 설명해주세요. 슬리피지 같은 잠재적 위험도 짧게 언급해주세요.";
            }
            
            try {
                const apiKey = ""; 
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                
                const payload = {
                    contents: [{ role: "user", parts: [{ text: prompt }] }]
                };

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error(`API 호출 실패: ${response.status}`);
                }
                
                const result = await response.json();

                if (result.candidates && result.candidates.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    textElement.textContent = text;
                } else {
                    throw new Error("AI로부터 유효한 응답을 받지 못했습니다.");
                }

            } catch (error) {
                console.error("Gemini API Error:", error);
                textElement.textContent = "AI 설명 생성 중 오류가 발생했습니다. 잠시 후 다시 시도해주세요.";
                button.disabled = false; // Allow retry on error
            } finally {
                loaderElement.classList.add('hidden');
                textElement.classList.remove('hidden');
            }
        };

        document.querySelectorAll('.ai-explain-btn').forEach(button => {
            button.addEventListener('click', (e) => {
                const method = e.target.dataset.method;
                getAiExplanation(method);
            });
        });
        // --- End Gemini AI Feature ---

        if (isFirebaseConfigured) {
            try {
                const { initializeApp } = await import("https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js");
                const { getAuth, signInAnonymously } = await import("https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js");
                const { getFirestore, doc, getDoc, addDoc, collection } = await import("https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js");

                const app = initializeApp(firebaseConfig);
                const auth = getAuth(app);
                const db = getFirestore(app);

                let userId = null;
                let currentGuideId = null;
                let shareableUrl = null;
                let userChoice = null;

                const copyLinkToClipboard = (url) => {
                    const urlToCopy = url || window.location.href;
                    const tempInput = document.createElement('textarea');
                    tempInput.value = urlToCopy;
                    document.body.appendChild(tempInput);
                    tempInput.select();
                    document.execCommand('copy');
                    document.body.removeChild(tempInput);
                    
                    toast.classList.remove('opacity-0');
                    setTimeout(() => {
                        toast.classList.add('opacity-0');
                    }, 3000);
                };

                shareBtn.addEventListener('click', async () => {
                    userChoice = redeemInfo.classList.contains('hidden') ? 'swap' : 'redeem';
                    if (!userChoice && !currentGuideId) return;

                    if (!currentGuideId) {
                        try {
                            shareBtn.disabled = true;
                            shareBtn.innerHTML = '<span>링크 생성 중...</span>';
                            const docRef = await addDoc(collection(db, `guides`), {
                                choice: userChoice,
                                creatorId: userId,
                                createdAt: new Date()
                            });
                            currentGuideId = docRef.id;
                            shareableUrl = `${window.location.href.split('?')[0]}?id=${currentGuideId}`;
                            
                            btnYes.disabled = true;
                            btnNo.disabled = true;

                        } catch (error) {
                            console.error("Error creating guide:", error);
                            errorDiv.textContent = '공유 링크 생성 중 오류가 발생했습니다.';
                            errorDiv.classList.add('hidden', 'text-red-500', 'bg-red-100');
                            shareBtn.disabled = false;
                            shareBtn.innerHTML = '이 페이지 공유하기 🔗';
                            return;
                        }
                    }
                    
                    shareBtn.disabled = false;
                    shareBtn.innerHTML = '이 페이지 공유하기 🔗';

                    const urlToShare = shareableUrl || window.location.href;
                    const shareData = {
                        title: 'AID 처분 방법 안내',
                        text: '당신에게 맞는 AID 처분 방법을 알아보세요!',
                        url: urlToShare
                    };

                    if (navigator.share) {
                        try {
                            await navigator.share(shareData);
                        } catch (err) {
                            if (err.name !== 'NotAllowedError') {
                               console.error("Share failed:", err);
                            }
                            copyLinkToClipboard(urlToShare);
                        }
                    } else {
                        copyLinkToClipboard(urlToShare);
                    }
                });
                
                auth.onAuthStateChanged(async (user) => {
                    if (user) {
                        userId = user.uid;
                        const urlParams = new URLSearchParams(window.location.search);
                        const guideIdFromUrl = urlParams.get('id');

                        if (guideIdFromUrl) {
                            currentGuideId = guideIdFromUrl;
                            shareableUrl = window.location.href;
                            try {
                                const docRef = doc(db, `guides`, currentGuideId);
                                const docSnap = await getDoc(docRef);

                                if (docSnap.exists()) {
                                    const data = docSnap.data();
                                    showInfo(data.choice);
                                    btnYes.disabled = true;
                                    btnNo.disabled = true;
                                } else {
                                    errorDiv.textContent = '요청하신 가이드를 찾을 수 없습니다.';
                                    errorDiv.classList.add('text-yellow-700', 'bg-yellow-100').remove('hidden');
                                }
                            } catch (error) {
                                 errorDiv.textContent = '가이드를 불러오는 중 오류가 발생했습니다.';
                                 errorDiv.classList.add('text-red-500', 'bg-red-100').remove('hidden');
                            }
                        }
                    }
                });

                signInAnonymously(auth).catch((error) => {
                     errorDiv.textContent = '익명 인증에 실패했습니다. 공유 기능을 사용할 수 없습니다.';
                     errorDiv.classList.add('text-red-500', 'bg-red-100').remove('hidden');
                });

            } catch(e) {
                errorDiv.textContent = 'Firebase 초기화 중 오류: ' + e.message;
                errorDiv.classList.add('text-red-500', 'bg-red-100').remove('hidden');
                shareBtn.style.display = 'none';
            }
        } else {
            // Firebase 구성이 없으면 공유 버튼만 숨깁니다.
            shareBtn.style.display = 'none';
        }

    </script>
</body>
</html>
