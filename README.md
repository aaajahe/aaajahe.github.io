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
    .modal-content { max-height: 85vh; }
    .loader {
      border: 4px solid #f3f3f3;
      border-top: 4px solid #3498db;
      border-radius: 50%;
      width: 24px;
      height: 24px;
      animation: spin 1s linear infinite;
    }
    @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
  </style>
</head>
<body class="bg-gray-50 text-gray-800">

<div id="app" class="container mx-auto p-4 md:p-6 max-w-7xl">
  <header class="text-center mb-8">
    <h1 class="text-4xl md:text-5xl font-bold text-gray-800">🚀 학습 허브</h1>
    <p class="text-gray-600 mt-2">매일 배우고, 함께 문제를 해결해요!</p>
    <div id="user-info" class="mt-4 text-sm text-gray-500 font-mono"></div>
  </header>

  <main class="grid grid-cols-1 lg:grid-cols-3 gap-6 md:gap-8">
    <div class="lg:col-span-1 space-y-6 md:space-y-8">
      <div class="bg-white p-6 rounded-2xl shadow-lg transition-shadow hover:shadow-xl">
        <h2 class="text-2xl font-bold mb-4 border-b pb-3 text-gray-700">오늘의 단어 📖</h2>
        <div id="word-of-the-day" class="space-y-3"></div>
      </div>

      <div class="bg-white p-6 rounded-2xl shadow-lg transition-shadow hover:shadow-xl">
        <h2 class="text-2xl font-bold mb-4 border-b pb-3 text-gray-700">질문 올리기 💬</h2>
        <form id="post-form" class="space-y-4">
          <div>
            <label for="post-title" class="block text-sm font-medium text-gray-700 mb-1">질문 제목</label>
            <input type="text" id="post-title" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" required placeholder="예: 이 수학 문제가 안 풀려요."/>
          </div>
          <div>
            <label for="post-image" class="block text-sm font-medium text-gray-700 mb-1">문제 사진</label>
            <input type="file" id="post-image" accept="image/*" class="mt-1 block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-600 hover:file:bg-indigo-100 cursor-pointer" required />
          </div>
          <button type="submit" id="submit-post-btn" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-4 rounded-lg flex items-center justify-center transition-all duration-300 disabled:bg-indigo-400 disabled:cursor-not-allowed shadow-md hover:shadow-lg" disabled>
            <span id="submit-btn-text">질문 등록하기</span>
            <div id="submit-loader" class="loader hidden"></div>
          </button>
        </form>
      </div>
    </div>

    <div class="lg:col-span-2 bg-white p-6 rounded-2xl shadow-lg transition-shadow hover:shadow-xl">
      <h2 class="text-2xl font-bold mb-4 border-b pb-3 text-gray-700">Q&A 포럼 🧠</h2>
      <div id="posts-container" class="space-y-4 h-[65vh] overflow-y-auto pr-2"></div>
    </div>
  </main>
</div>

<div id="post-modal" class="fixed inset-0 bg-black bg-opacity-60 hidden items-center justify-center p-4 z-50">
  <div class="bg-white rounded-2xl shadow-xl w-full max-w-4xl modal-content overflow-hidden flex flex-col">
    <div class="p-5 border-b flex justify-between items-center bg-gray-50">
      <h3 id="modal-title" class="text-xl md:text-2xl font-bold text-gray-800"></h3>
      <button id="close-modal-btn" class="text-gray-500 hover:text-gray-800 text-3xl leading-none">&times;</button>
    </div>
    <div class="p-6 overflow-y-auto flex-grow">
      <div class="text-sm text-gray-500 mb-4">
        <span>작성자: </span>
        <span id="modal-author" class="font-mono bg-gray-100 px-2 py-1 rounded"></span>
      </div>
      <div class="mb-6 bg-gray-100 rounded-lg flex items-center justify-center p-4">
        <img id="modal-image" src="" alt="질문 이미지" class="w-full h-auto object-contain rounded-lg max-h-[40vh]" />
      </div>
      
      <h4 class="text-xl font-semibold mb-4 border-b pb-2">토론</h4>
      <div id="comments-container" class="space-y-4 mb-6 max-h-48 overflow-y-auto pr-2"></div>
      <form id="comment-form">
        <input type="hidden" id="current-post-id" />
        <div class="flex gap-2 items-center">
          <input type="text" id="comment-input" class="flex-grow rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" placeholder="의견을 남겨주세요..." required />
          <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-lg transition-colors">등록</button>
        </div>
      </form>
    </div>
  </div>
</div>

<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
  import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
  import { getFirestore, collection, addDoc, doc, onSnapshot, getDoc, query, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
  import { getStorage, ref, uploadBytes, getDownloadURL } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-storage.js";

  const firebaseConfig = typeof __firebase_config !== 'undefined' 
    ? JSON.parse(__firebase_config)
    : { apiKey: "YOUR_API_KEY", authDomain: "YOUR_AUTH_DOMAIN", projectId: "YOUR_PROJECT_ID", storageBucket: "YOUR_STORAGE_BUCKET" };

  const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
  const app = initializeApp(firebaseConfig);
  const auth = getAuth(app);
  const db = getFirestore(app);
  const storage = getStorage(app);

  let userId = null;
  let commentsUnsubscribe = null;

  const userInfoDiv = document.getElementById('user-info');
  const wordContainer = document.getElementById('word-of-the-day');
  const postsContainer = document.getElementById('posts-container');
  const postForm = document.getElementById('post-form');
  const postTitleInput = document.getElementById('post-title');
  const postImageInput = document.getElementById('post-image');
  const submitPostBtn = document.getElementById('submit-post-btn');
  const submitBtnText = document.getElementById('submit-btn-text');
  const submitLoader = document.getElementById('submit-loader');

  const modal = document.getElementById('post-modal');
  const modalTitle = document.getElementById('modal-title');
  const modalAuthor = document.getElementById('modal-author');
  const modalImage = document.getElementById('modal-image');
  const closeModalBtn = document.getElementById('close-modal-btn');
  const commentsContainer = document.getElementById('comments-container');
  const commentForm = document.getElementById('comment-form');
  const commentInput = document.getElementById('comment-input');
  const currentPostIdInput = document.getElementById('current-post-id');

  function displayWordOfTheDay() {
    const words = [
      { word: "Collaborate", meaning: "협력하다", sentence: "Let's collaborate on this project to achieve better results." },
      { word: "Resilience", meaning: "회복력", sentence: "The team showed great resilience after a tough loss." },
      { word: "Innovate", meaning: "혁신하다", sentence: "We must constantly innovate to stay ahead of the competition." },
      { word: "Persistence", meaning: "끈기", sentence: "Her persistence paid off when she finally solved the puzzle." },
      { word: "Empathy", meaning: "공감", sentence: "Empathy is key to understanding others' perspectives." }
    ];
    const index = new Date().getDate() % words.length;
    const word = words[index];
    wordContainer.innerHTML = `
      <p class="text-3xl font-semibold text-indigo-600">${word.word}</p>
      <p class="text-gray-600 mt-1">${word.meaning}</p>
      <p class="text-gray-500 italic mt-3 text-sm">"${word.sentence}"</p>
    `;
  }

  function fetchAndRenderPosts() {
    const postsCollectionPath = `/artifacts/${appId}/public/data/posts`;
    const q = query(collection(db, postsCollectionPath));

    onSnapshot(q, (snapshot) => {
      const posts = [];
      snapshot.forEach(doc => posts.push({ id: doc.id, ...doc.data() }));
      posts.sort((a, b) => (b.createdAt?.toMillis() || 0) - (a.createdAt?.toMillis() || 0));
      postsContainer.innerHTML = '';
      if (posts.length === 0) {
        postsContainer.innerHTML = `<p class="text-center text-gray-500 mt-8">아직 질문이 없습니다. 첫 질문을 올려보세요!</p>`;
      } else {
        posts.forEach(post => {
          const postEl = document.createElement('div');
          postEl.className = 'p-4 border rounded-xl hover:shadow-lg hover:border-indigo-300 cursor-pointer transition-all duration-200 bg-white';
          postEl.innerHTML = `
            <h3 class="font-bold text-lg text-gray-800 truncate">${post.title}</h3>
            <p class="text-sm text-gray-500 mt-1">작성자: <span class="font-mono text-xs">${post.authorId}</span></p>
            <p class="text-xs text-gray-400 mt-2">${post.createdAt ? new Date(post.createdAt.toMillis()).toLocaleString() : '방금 전'}</p>
          `;
          postEl.addEventListener('click', () => openPostModal(post.id));
          postsContainer.appendChild(postEl);
        });
      }
    });
  }

  async function openPostModal(postId) {
    const postDocPath = `/artifacts/${appId}/public/data/posts/${postId}`;
    const docSnap = await getDoc(doc(db, postDocPath));
    if (!docSnap.exists()) {
      console.error("게시물을 찾을 수 없습니다.");
      return;
    }

    const data = docSnap.data();
    modalTitle.textContent = data.title;
    modalAuthor.textContent = data.authorId;
    modalImage.src = data.imageUrl;
    modalImage.alt = data.title;
    currentPostIdInput.value = postId;

    modal.classList.remove('hidden');
    modal.classList.add('flex');
    document.body.style.overflow = 'hidden';

    if (commentsUnsubscribe) commentsUnsubscribe();
    fetchAndRenderComments(postId);
  }

  function closePostModal() {
    modal.classList.add('hidden');
    modal.classList.remove('flex');
    document.body.style.overflow = 'auto';
    if (commentsUnsubscribe) {
      commentsUnsubscribe();
      commentsUnsubscribe = null;
    }
    commentsContainer.innerHTML = '';
  }

  function fetchAndRenderComments(postId) {
    const commentsCollectionPath = `/artifacts/${appId}/public/data/posts/${postId}/comments`;
    const q = query(collection(db, commentsCollectionPath));

    commentsUnsubscribe = onSnapshot(q, (snapshot) => {
      const comments = [];
      snapshot.forEach(doc => comments.push({ id: doc.id, ...doc.data() }));
      comments.sort((a, b) => (a.createdAt?.toMillis() || 0) - (b.createdAt?.toMillis() || 0));
      commentsContainer.innerHTML = '';
      if (comments.length === 0) {
        commentsContainer.innerHTML = `<p class="text-center text-sm text-gray-400">아직 댓글이 없습니다.</p>`;
      } else {
        comments.forEach(comment => {
          const commentEl = document.createElement('div');
          commentEl.className = 'p-3 bg-gray-100 rounded-lg';
          commentEl.innerHTML = `
            <p class="text-sm">${comment.text}</p>
            <p class="text-xs text-gray-500 mt-1 text-right font-mono">${comment.authorId.substring(0, 12)}...</p>
          `;
          commentsContainer.appendChild(commentEl);
        });
        commentsContainer.scrollTop = commentsContainer.scrollHeight;
      }
    });
  }

  postForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    const title = postTitleInput.value.trim();
    const imageFile = postImageInput.files[0];

    if (!title || !imageFile || !userId) {
      alert("질문 제목과 이미지를 모두 입력해 주세요.");
      return;
    }

    submitPostBtn.disabled = true;
    submitBtnText.classList.add('hidden');
    submitLoader.classList.remove('hidden');

    try {
      const filePath = `artifacts/${appId}/uploads/${Date.now()}_${imageFile.name}`;
      const storageRef = ref(storage, filePath);
      const uploadResult = await uploadBytes(storageRef, imageFile);
      const imageUrl = await getDownloadURL(uploadResult.ref);

      const postsCollectionPath = `/artifacts/${appId}/public/data/posts`;
      await addDoc(collection(db, postsCollectionPath), {
        title: title,
        imageUrl: imageUrl,
        authorId: userId,
        createdAt: serverTimestamp()
      });

      postForm.reset();
      alert("🎉 질문이 성공적으로 등록되었습니다!");

    } catch (error) {
      console.error("게시물 등록 실패:", error);
      alert("⚠️ 질문 등록에 실패했습니다. 네트워크 또는 권한 문제일 수 있습니다.");
    } finally {
      submitPostBtn.disabled = false;
      submitBtnText.classList.remove('hidden');
      submitLoader.classList.add('hidden');
    }
  });

  commentForm.addEventListener('submit', async (e) => {
    e.preventDefault();
    const postId = currentPostIdInput.value;
    const text = commentInput.value.trim();
    if (!text || !postId || !userId) return;

    const commentsCollectionPath = `/artifacts/${appId}/public/data/posts/${postId}/comments`;
    try {
      await addDoc(collection(db, commentsCollectionPath), {
        text: text,
        authorId: userId,
        createdAt: serverTimestamp()
      });
      commentForm.reset();
    } catch (error) {
      console.error("댓글 등록 실패:", error);
    }
  });

  closeModalBtn.addEventListener('click', closePostModal);
  modal.addEventListener('click', (e) => {
    if (e.target === modal) closePostModal();
  });

  onAuthStateChanged(auth, async (user) => {
    if (user) {
      userId = user.uid;
      userInfoDiv.innerHTML = `로그인 ID: <span class="font-semibold">${userId}</span> (익명)`;
      fetchAndRenderPosts();
      submitPostBtn.disabled = false;
    } else {
      try {
        const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        if (token) await signInWithCustomToken(auth, token);
        else await signInAnonymously(auth);
      } catch (error) {
        console.error("익명 로그인 실패:", error);
        userInfoDiv.textContent = "인증에 실패했습니다. 새로고침 해주세요.";
      }
    }
  });

  displayWordOfTheDay();
</script>

</body>
</html>
