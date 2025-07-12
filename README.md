<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>협업 학습 게시판</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', 'Noto Sans KR', sans-serif;
        }
        /* 스크롤바 스타일링 */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        ::-webkit-scrollbar-thumb {
            background: #888;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #555;
        }
        .modal-content {
            max-height: 80vh;
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3498db;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div id="app" class="container mx-auto p-4 md:p-8 max-w-7xl">
        <header class="text-center mb-8">
            <h1 class="text-4xl md:text-5xl font-bold text-gray-800">🚀 학습 허브</h1>
            <p class="text-gray-600 mt-2">매일 배우고, 함께 해결해요!</p>
            <div id="user-info" class="mt-4 text-sm text-gray-500"></div>
        </header>

        <main class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- 왼쪽: 오늘의 단어 및 질문 올리기 -->
            <div class="lg:col-span-1 space-y-8">
                <!-- 오늘의 단어 -->
                <div class="bg-white p-6 rounded-2xl shadow-lg">
                    <h2 class="text-2xl font-bold mb-4 border-b pb-2">오늘의 단어 📖</h2>
                    <div id="word-of-the-day" class="space-y-3">
                        <!-- JS로 내용 채움 -->
                    </div>
                </div>

                <!-- 질문 올리기 -->
                <div class="bg-white p-6 rounded-2xl shadow-lg">
                    <h2 class="text-2xl font-bold mb-4 border-b pb-2">질문 올리기 💬</h2>
                    <form id="post-form" class="space-y-4">
                        <div>
                            <label for="post-title" class="block text-sm font-medium text-gray-700">질문 제목</label>
                            <input type="text" id="post-title" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" placeholder="예: 이 수학 문제 풀이가 궁금해요!" required>
                        </div>
                        <div>
                            <label for="post-image" class="block text-sm font-medium text-gray-700">문제 사진</label>
                            <input type="file" id="post-image" accept="image/*" class="mt-1 block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-600 hover:file:bg-indigo-100" required>
                        </div>
                        <button type="submit" id="submit-post-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg flex items-center justify-center transition-colors">
                            <span id="submit-btn-text">질문 등록하기</span>
                            <div id="submit-loader" class="loader hidden"></div>
                        </button>
                    </form>
                </div>
            </div>

            <!-- 오른쪽: Q&A 포럼 -->
            <div class="lg:col-span-2 bg-white p-6 rounded-2xl shadow-lg">
                <h2 class="text-2xl font-bold mb-4 border-b pb-2">Q&A 포럼 🧠</h2>
                <div id="posts-container" class="space-y-4 h-[60vh] overflow-y-auto pr-2">
                    <!-- JS로 내용 채움 -->
                </div>
            </div>
        </main>
    </div>

    <!-- 상세 보기 모달 -->
    <div id="post-modal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-xl w-full max-w-3xl modal-content overflow-hidden flex flex-col">
            <div class="p-6 border-b flex justify-between items-center">
                <h3 id="modal-title" class="text-2xl font-bold"></h3>
                <button id="close-modal-btn" class="text-gray-500 hover:text-gray-800">&times;</button>
            </div>
            <div class="p-6 overflow-y-auto flex-grow">
                <p class="text-sm text-gray-500 mb-4">작성자: <span id="modal-author"></span></p>
                <img id="modal-image" src="" alt="문제 이미지" class="w-full h-auto object-contain rounded-lg mb-6 max-h-96">
                
                <h4 class="text-xl font-semibold mb-4">토론</h4>
                <div id="comments-container" class="space-y-4 mb-6">
                    <!-- JS로 댓글 채움 -->
                </div>
                <form id="comment-form">
                    <input type="hidden" id="current-post-id">
                    <div class="flex gap-2">
                        <input type="text" id="comment-input" class="flex-grow block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" placeholder="의견을 남겨주세요..." required>
                        <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg transition-colors">등록</button>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <script type="module">
        // Firebase SDK import
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, doc, onSnapshot, query, setDoc, getDoc, updateDoc, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getStorage, ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-storage.js";

        // Firebase 설정
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : { apiKey: "YOUR_API_KEY", authDomain: "YOUR_AUTH_DOMAIN", projectId: "YOUR_PROJECT_ID" };
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        // Firebase 앱 초기화
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const storage = getStorage(app);

        let currentUser = null;
        let userId = null;

        // 오늘의 단어 데이터
        const words = [
            { word: "Collaborate", meaning: "협력하다", sentence: "Let's collaborate on this project to achieve better results." },
            { word: "Resilience", meaning: "회복력", sentence: "The team showed great resilience after a tough loss." },
            { word: "Innovate", meaning: "혁신하다", sentence: "We must constantly innovate to stay ahead of the competition." },
            { word: "Empathy", meaning: "공감", sentence: "Having empathy for others is a key leadership quality." },
            { word: "Persevere", meaning: "인내하며 계속하다", sentence: "Despite the difficulties, she persevered and achieved her goal." },
        ];

        // UI 요소
        const wordContainer = document.getElementById('word-of-the-day');
        const postsContainer = document.getElementById('posts-container');
        const postForm = document.getElementById('post-form');
        const submitPostBtn = document.getElementById('submit-post-btn');
        const submitBtnText = document.getElementById('submit-btn-text');
        const submitLoader = document.getElementById('submit-loader');
        const userInfoDiv = document.getElementById('user-info');

        // 모달 UI 요소
        const modal = document.getElementById('post-modal');
        const closeModalBtn = document.getElementById('close-modal-btn');
        const modalTitle = document.getElementById('modal-title');
        const modalAuthor = document.getElementById('modal-author');
        const modalImage = document.getElementById('modal-image');
        const commentsContainer = document.getElementById('comments-container');
        const commentForm = document.getElementById('comment-form');
        const commentInput = document.getElementById('comment-input');
        const currentPostIdInput = document.getElementById('current-post-id');

        // 오늘의 단어 표시
        function displayWordOfTheDay() {
            const today = new Date();
            const dayIndex = today.getDate() % words.length;
            const wordData = words[dayIndex];

            wordContainer.innerHTML = `
                <p class="text-3xl font-semibold text-indigo-600">${wordData.word}</p>
                <p class="text-gray-600">${wordData.meaning}</p>
                <p class="text-gray-500 italic">"${wordData.sentence}"</p>
            `;
        }

        // 포스트 렌더링
        function renderPost(post) {
            const postEl = document.createElement('div');
            postEl.className = 'p-4 border rounded-lg hover:shadow-md transition-shadow cursor-pointer';
            postEl.dataset.id = post.id;
            
            const postDate = post.data.createdAt?.toDate().toLocaleString('ko-KR') || '날짜 정보 없음';

            postEl.innerHTML = `
                <h3 class="font-bold text-lg">${post.data.title}</h3>
                <p class="text-sm text-gray-500">작성자: ${post.data.authorId.substring(0, 8)}...</p>
                <p class="text-sm text-gray-400">${postDate}</p>
            `;
            
            postEl.addEventListener('click', () => openPostModal(post.id));
            
            // 최신 글이 위로 오도록
            if (postsContainer.firstChild) {
                postsContainer.insertBefore(postEl, postsContainer.firstChild);
            } else {
                postsContainer.appendChild(postEl);
            }
        }

        // 포스트 불러오기
        async function fetchPosts() {
            const postsCollectionPath = `/artifacts/${appId}/public/data/posts`;
            const q = query(collection(db, postsCollectionPath), orderBy("createdAt", "desc"));
            
            onSnapshot(q, (querySnapshot) => {
                postsContainer.innerHTML = ''; // 기존 목록 초기화
                querySnapshot.forEach((doc) => {
                    renderPost({ id: doc.id, data: doc.data() });
                });
            });
        }

        // 포스트 등록 처리
        postForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            if (!currentUser) {
                alert("로그인이 필요합니다.");
                return;
            }

            const title = document.getElementById('post-title').value;
            const imageFile = document.getElementById('post-image').files[0];

            if (!title || !imageFile) {
                alert("제목과 이미지 파일을 모두 선택해주세요.");
                return;
            }

            submitPostBtn.disabled = true;
            submitBtnText.classList.add('hidden');
            submitLoader.classList.remove('hidden');

            try {
                // 1. 이미지 Firebase Storage에 업로드
                // 공개 접근 규칙에 맞게 스토리지 경로를 수정했습니다.
                const storageRef = ref(storage, `artifacts/${appId}/public/images/${Date.now()}_${imageFile.name}`);
                const snapshot = await uploadBytes(storageRef, imageFile);
                const imageUrl = await getDownloadURL(snapshot.ref);

                // 2. Firestore에 포스트 데이터 저장
                const postsCollectionPath = `/artifacts/${appId}/public/data/posts`;
                await addDoc(collection(db, postsCollectionPath), {
                    title: title,
                    imageUrl: imageUrl,
                    authorId: userId,
                    createdAt: serverTimestamp()
                });

                postForm.reset();
            } catch (error) {
                console.error("Error adding document: ", error);
                alert("질문 등록에 실패했습니다.");
            } finally {
                submitPostBtn.disabled = false;
                submitBtnText.classList.remove('hidden');
                submitLoader.classList.add('hidden');
            }
        });

        // 모달 열기
        async function openPostModal(postId) {
            currentPostIdInput.value = postId;
            const postDocPath = `/artifacts/${appId}/public/data/posts/${postId}`;
            const postDoc = await getDoc(doc(db, postDocPath));

            if (postDoc.exists()) {
                const postData = postDoc.data();
                modalTitle.textContent = postData.title;
                modalAuthor.textContent = postData.authorId;
                modalImage.src = postData.imageUrl;
                modal.classList.remove('hidden');
                modal.classList.add('flex');
                
                fetchComments(postId);
            }
        }

        // 모달 닫기
        function closePostModal() {
            modal.classList.add('hidden');
            modal.classList.remove('flex');
            commentsContainer.innerHTML = ''; // 댓글 내용 초기화
        }
        closeModalBtn.addEventListener('click', closePostModal);
        modal.addEventListener('click', (e) => {
            if (e.target === modal) {
                closePostModal();
            }
        });

        // 댓글 불러오기
        function fetchComments(postId) {
            const commentsCollectionPath = `/artifacts/${appId}/public/data/posts/${postId}/comments`;
            const q = query(collection(db, commentsCollectionPath), orderBy("createdAt"));

            onSnapshot(q, (querySnapshot) => {
                commentsContainer.innerHTML = '';
                if (querySnapshot.empty) {
                    commentsContainer.innerHTML = '<p class="text-gray-500">아직 댓글이 없습니다. 첫 댓글을 남겨보세요!</p>';
                } else {
                    querySnapshot.forEach((doc) => {
                        renderComment(doc.data());
                    });
                }
            });
        }
        
        // 댓글 렌더링
        function renderComment(commentData) {
            const commentEl = document.createElement('div');
            commentEl.className = 'p-3 bg-gray-100 rounded-lg';
            const commentDate = commentData.createdAt?.toDate().toLocaleString('ko-KR') || '';
            
            commentEl.innerHTML = `
                <p class="text-sm">${commentData.text}</p>
                <p class="text-xs text-gray-500 mt-1">작성자: ${commentData.authorId.substring(0,8)}... - ${commentDate}</p>
            `;
            commentsContainer.appendChild(commentEl);
        }

        // 댓글 등록 처리
        commentForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const postId = currentPostIdInput.value;
            const text = commentInput.value;

            if (!text.trim() || !currentUser) return;
            
            const commentsCollectionPath = `/artifacts/${appId}/public/data/posts/${postId}/comments`;
            try {
                await addDoc(collection(db, commentsCollectionPath), {
                    text: text,
                    authorId: userId,
                    createdAt: serverTimestamp()
                });
                commentInput.value = '';
            } catch (error) {
                console.error("Error adding comment: ", error);
                alert("댓글 등록에 실패했습니다.");
            }
        });


        // 인증 상태 리스너
        onAuthStateChanged(auth, async (user) => {
            if (user) {
                currentUser = user;
                userId = user.uid;
                userInfoDiv.innerHTML = `로그인된 사용자: <span class="font-semibold">${userId}</span>`;
                fetchPosts();
            } else {
                try {
                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                } catch (error) {
                    console.error("Authentication failed:", error);
                    userInfoDiv.textContent = "인증에 실패했습니다. 새로고침 해주세요.";
                }
            }
        });

        // 앱 초기화 함수
        function init() {
            displayWordOfTheDay();
        }

        init();
    </script>
</body>
</html>
