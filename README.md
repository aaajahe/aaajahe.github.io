<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>협업 학습 게시판</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet" />
  <style>
    body { font-family: 'Inter', 'Noto Sans KR', sans-serif; }
    ::-webkit-scrollbar { width: 8px; }
    ::-webkit-scrollbar-track { background: #f1f1f1; }
    ::-webkit-scrollbar-thumb { background: #888; border-radius: 4px; }
    ::-webkit-scrollbar-thumb:hover { background: #555; }
    .modal-content { max-height: 80vh; }
    .loader {
      border: 4px solid #f3f3f3;
      border-top: 4px solid #3498db;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      animation: spin 1s linear infinite;
    }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
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
    <div class="lg:col-span-1 space-y-8">
      <!-- 오늘의 단어 -->
      <div class="bg-white p-6 rounded-2xl shadow-lg">
        <h2 class="text-2xl font-bold mb-4 border-b pb-2">오늘의 단어 📖</h2>
        <div id="word-of-the-day" class="space-y-3"></div>
      </div>

      <!-- 질문 올리기 -->
      <div class="bg-white p-6 rounded-2xl shadow-lg">
        <h2 class="text-2xl font-bold mb-4 border-b pb-2">질문 올리기 💬</h2>
        <form id="post-form" class="space-y-4">
          <div>
            <label for="post-title" class="block text-sm font-medium text-gray-700">질문 제목</label>
            <input type="text" id="post-title" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" required />
          </div>
          <div>
            <label for="post-image" class="block text-sm font-medium text-gray-700">문제 사진</label>
            <input type="file" id="post-image" accept="image/*" class="mt-1 block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-600 hover:file:bg-indigo-100" required />
          </div>
          <button type="submit" id="submit-post-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg flex items-center justify-center transition-colors disabled:bg-indigo-400 disabled:cursor-not-allowed" disabled>
            <span id="submit-btn-text">질문 등록하기</span>
            <div id="submit-loader" class="loader hidden"></div>
          </button>
        </form>
      </div>
    </div>

    <!-- Q&A 포럼 -->
    <div class="lg:col-span-2 bg-white p-6 rounded-2xl shadow-lg">
      <h2 class="text-2xl font-bold mb-4 border-b pb-2">Q&A 포럼 🧠</h2>
      <div id="posts-container" class="space-y-4 h-[60vh] overflow-y-auto pr-2"></div>
    </div>
  </main>
</div>

<!-- 모달 -->
<div id="post-modal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center p-4">
  <div class="bg-white rounded-2xl shadow-xl w-full max-w-3xl modal-content overflow-hidden flex flex-col">
    <div class="p-6 border-b flex justify-between items-center">
      <h3 id="modal-title" class="text-2xl font-bold"></h3>
      <button id="close-modal-btn" class="text-gray-500 hover:text-gray-800">&times;</button>
    </div>
    <div class="p-6 overflow-y-auto flex-grow">
      <p class="text-sm text-gray-500 mb-4">작성자: <span id="modal-author"></span></p>
      <img id="modal-image" src="" class="w-full h-auto object-contain rounded-lg mb-6 max-h-96" />
      <h4 class="text-xl font-semibold mb-4">토론</h4>
      <div id="comments-container" class="space-y-4 mb-6"></div>
      <form id="comment-form">
        <input type="hidden" id="current-post-id" />
        <div class="flex gap-2">
          <input type="text" id="comment-input" class="flex-grow rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" placeholder="의견을 남겨주세요..." required />
          <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg">등록</button>
        </div>
      </form>
    </div>
  </div>
</div>

<!-- Firebase SDK 및 JavaScript -->
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
  import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
  import { getFirestore, collection, addDoc, doc, onSnapshot, getDoc, query, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
  import { getStorage, ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-storage.js";

  // 🔐 Firebase 설정
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_AUTH_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_BUCKET.appspot.com"
  };

  const app = initializeApp(firebaseConfig);
  const auth = getAuth(app);
  const db = getFirestore(app);
  const storage = getStorage(app);

  let userId = null;

  // 오늘의 단어
  const words = [
    { word: "Collaborate", meaning: "협력하다", sentence: "Let's collaborate on this project to achieve better results." },
    { word: "Resilience", meaning: "회복력", sentence: "The team showed great resilience after a tough loss." },
    { word: "Innovate", meaning: "혁신하다", sentence: "We must constantly innovate to stay ahead of the competition." }
  ];

  // UI
  const wordContainer = document.getElementById('word-of-the-day');
  const postsContainer = document.getElementById('posts-container');
  const postForm = document.getElementById('post-form');
  const submitPostBtn = document.getElementById('submit-post-btn');
  const submitBtnText = document.getElementById('submit-btn-text');
  const submitLoader = document.getElementById('submit-loader');
  const userInfoDiv = document.getElementById('user-info');
  const modal = document.getElementById('post-modal');
  const modalTitle = document.getElementById('modal-title');
  const modalAuthor = document.getElementById('modal-author');
  const modalImage = document.getElementById('modal-image');
  const commentsContainer = document.getElementById('comments-container');
  const commentForm = document.getElementById('comment-form');
  const commentInput = document.getElementById('comment-input');
  const currentPostIdInput = document.getElementById('current-post-id');
  const closeModalBtn = document.getElementById('close-modal-btn');

  function displayWordOfTheDay() {
    const index = new Date().getDate() % words.length;
    const word = words[index];
    wordContainer.innerHTML = `
      <p class="text-3xl font-semibold text-indigo-600">${word.word}</p>
      <p class="text-gray-600">${word.meaning}</p>
      <p class="text-gray-500 italic">"${word.sentence}"</p>
    `;
  }

  function renderPost(post) {
    const el = document.createElement('div');
    el.className = 'p-4 border rounded-lg hover:shadow-md cursor-pointer';
    el.innerHTML = `
      <h3 class="font-bold text-lg">${post.data.title}</h3>
      <p class="text-sm text-gray-500">작성자: ${post.data.authorId.slice(0, 8)}...</p>
    `;
    el.addEventListener('click', () => openPostModal(post.id));
    postsContainer.prepend(el);
  }

  async function fetchPosts() {
    const q = query(collection(db, 'anonymous_uploads'), orderBy('createdAt', 'desc'));
    onSnapshot(q, (snapshot) => {
      postsContainer.innerHTML = '';
      snapshot.forEach((doc) => renderPost({ id: doc.id, data: doc.data() }));
    });
  }

  postForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    const title = document.getElementById('post-title').value;
    const image = document.getElementById('post-image').files[0];
    if (!title || !image) return;

    submitPostBtn.disabled = true;
    submitBtnText.classList.add('hidden');
    submitLoader.classList.remove('hidden');

    try {
      const storageRef = ref(storage, `uploads/${Date.now()}_${image.name}`);
      const snap = await uploadBytes(storageRef, image);
      const imageUrl = await getDownloadURL(snap.ref);

      await addDoc(collection(db, 'anonymous_uploads'), {
        title,
        imageUrl,
        authorId: userId,
        createdAt: serverTimestamp()
      });

      postForm.reset();
    } catch (err) {
      alert("등록 실패: " + err.message);
    } finally {
      submitPostBtn.disabled = false;
      submitBtnText.classList.remove('hidden');
      submitLoader.classList.add('hidden');
    }
  });

  async function openPostModal(postId) {
    currentPostIdInput.value = postId;
    const docSnap = await getDoc(doc(db, 'anonymous_uploads', postId));
    if (!docSnap.exists()) return;
    const data = docSnap.data();
    modalTitle.textContent = data.title;
    modalAuthor.textContent = data.authorId;
    modalImage.src = data.imageUrl;
    modal.classList.remove('hidden');
    modal.classList.add('flex');
  }

  closeModalBtn.onclick = () => modal.classList.add('hidden');
  modal.onclick = (e) => { if (e.target === modal) modal.classList.add('hidden'); };

  onAuthStateChanged(auth, async (user) => {
    if (user) {
      userId = user.uid;
      userInfoDiv.innerHTML = `익명으로 사용 중입니다. (ID: <span class="font-semibold">${userId}</span>)`;
      fetchPosts();
      submitPostBtn.disabled = false;
    } else {
      await signInAnonymously(auth);
    }
  });

  displayWordOfTheDay();
</script>
</body>
</html>
