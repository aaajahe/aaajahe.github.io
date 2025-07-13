<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>파이썬/자바스크립트 사진첩</title>
    <!-- Tailwind CSS 로드 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 기본 폰트 및 부드러운 스크롤 설정 */
        body {
            font-family: 'Inter', sans-serif;
            scroll-behavior: smooth;
        }
    </style>
    <link rel="stylesheet" href="https://rsms.me/inter/inter.css">
</head>
<body class="bg-gray-100">

    <div class="container mx-auto max-w-3xl p-4">
        <header class="text-center my-8">
            <h1 class="text-4xl font-bold text-gray-800">학습 포럼</h1>
            <p class="text-gray-500 mt-2">글과 사진을 올리고 자유롭게 소통해보세요.</p>
        </header>

        <!-- 새 게시글 작성 폼 -->
        <div class="bg-white p-6 rounded-lg shadow-md mb-8">
            <h2 class="text-2xl font-semibold mb-4">새 글 작성하기</h2>
            <form id="new-post-form">
                <div class="mb-4">
                    <label for="author" class="block text-gray-700 font-medium mb-2">이름</label>
                    <input type="text" id="author" name="author" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required>
                </div>
                <div class="mb-4">
                    <label for="text_content" class="block text-gray-700 font-medium mb-2">내용</label>
                    <textarea id="text_content" name="text_content" rows="4" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500" required></textarea>
                </div>
                <div class="mb-4">
                    <label for="image_path" class="block text-gray-700 font-medium mb-2">이미지 URL (선택 사항)</label>
                    <input type="text" id="image_path" name="image_path" placeholder="https://example.com/image.jpg" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <button type="submit" class="w-full bg-blue-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-700 transition duration-300">
                    게시하기
                </button>
            </form>
        </div>

        <!-- 게시글 목록이 표시될 영역 -->
        <main id="posts-container" class="space-y-8">
            <!-- 자바스크립트로 게시글이 여기에 채워집니다. -->
        </main>
    </div>

    <script>
        const API_URL = 'http://127.0.0.1:5000'; // 파이썬 백엔드 서버 주소

        // --- API 호출 함수 ---

        // 모든 게시글 가져오기
        async function fetchPosts() {
            try {
                const response = await fetch(`${API_URL}/api/posts`);
                if (!response.ok) throw new Error('서버에서 데이터를 가져오는 데 실패했습니다.');
                const posts = await response.json();
                renderPosts(posts);
            } catch (error) {
                console.error('Fetch Error:', error);
                document.getElementById('posts-container').innerHTML = `<p class="text-center text-red-500">${error.message}</p>`;
            }
        }

        // 새 게시글 생성하기
        async function createPost(postData) {
            try {
                const response = await fetch(`${API_URL}/api/posts`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(postData),
                });
                if (!response.ok) throw new Error('게시글 작성에 실패했습니다.');
                await response.json();
                fetchPosts(); // 목록 새로고침
            } catch (error) {
                console.error('Create Post Error:', error);
                alert(error.message);
            }
        }

        // 새 댓글 추가하기
        async function addComment(postId, commentData) {
            try {
                const response = await fetch(`${API_URL}/api/posts/${postId}/comments`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(commentData),
                });
                if (!response.ok) throw new Error('댓글 작성에 실패했습니다.');
                await response.json();
                fetchPosts(); // 목록 새로고침
            } catch (error) {
                console.error('Add Comment Error:', error);
                alert(error.message);
            }
        }

        // --- 렌더링 함수 ---

        // 게시글 목록을 화면에 그리기
        function renderPosts(posts) {
            const container = document.getElementById('posts-container');
            container.innerHTML = ''; // 기존 내용을 비움

            if (posts.length === 0) {
                container.innerHTML = '<p class="text-center text-gray-500">아직 게시글이 없습니다. 첫 글을 작성해보세요!</p>';
                return;
            }

            posts.forEach(post => {
                const postElement = document.createElement('article');
                postElement.className = 'bg-white p-6 rounded-lg shadow-md';
                
                let imageHTML = '';
                // 이미지 경로가 있을 경우에만 img 태그를 추가
                if (post.image_path) {
                    imageHTML = `<img src="${post.image_path}" alt="게시물 이미지" class="mt-4 rounded-lg w-full h-auto object-cover" onerror="this.style.display='none'">`;
                }

                postElement.innerHTML = `
                    <div class="flex items-center mb-4">
                        <div class="w-10 h-10 rounded-full bg-gray-300 flex items-center justify-center font-bold text-gray-600 mr-4">${post.author.charAt(0)}</div>
                        <div>
                            <h3 class="font-bold text-lg text-gray-800">${post.author}</h3>
                            <p class="text-sm text-gray-500">Post ID: ${post.post_id}</p>
                        </div>
                    </div>
                    <p class="text-gray-700 whitespace-pre-wrap">${post.text_content}</p>
                    ${imageHTML}
                    <div class="mt-4 border-t pt-4">
                        <h4 class="font-semibold text-gray-600 mb-2">댓글</h4>
                        <div class="comments-list space-y-2">
                            ${post.comments.map(c => `<div class="bg-gray-100 p-2 rounded-lg"><strong class="text-gray-800">${c.author}:</strong> ${c.content}</div>`).join('') || '<p class="text-gray-400 text-sm">아직 댓글이 없습니다.</p>'}
                        </div>
                        <form class="comment-form mt-4">
                            <input type="hidden" name="post_id" value="${post.post_id}">
                            <input type="text" name="comment_author" placeholder="이름" class="w-full mb-2 px-3 py-1 border rounded-md text-sm" required>
                            <input type="text" name="comment_content" placeholder="댓글을 입력하세요..." class="w-full px-3 py-1 border rounded-md text-sm" required>
                            <button type="submit" class="mt-2 bg-gray-200 text-gray-700 text-sm font-semibold py-1 px-3 rounded-md hover:bg-gray-300">댓글 달기</button>
                        </form>
                    </div>
                `;
                container.appendChild(postElement);
            });
        }

        // --- 이벤트 리스너 설정 ---

        document.addEventListener('DOMContentLoaded', () => {
            const newPostForm = document.getElementById('new-post-form');
            const postsContainer = document.getElementById('posts-container');

            // 새 게시글 폼 제출 이벤트
            newPostForm.addEventListener('submit', (e) => {
                e.preventDefault();
                const author = e.target.author.value;
                const text_content = e.target.text_content.value;
                const image_path = e.target.image_path.value;
                
                createPost({ author, text_content, image_path });
                e.target.reset(); // 폼 초기화
            });

            // 댓글 폼 제출 이벤트 (이벤트 위임 사용)
            postsContainer.addEventListener('submit', (e) => {
                if (e.target.classList.contains('comment-form')) {
                    e.preventDefault();
                    const postId = e.target.post_id.value;
                    const author = e.target.comment_author.value;
                    const content = e.target.comment_content.value;
                    addComment(postId, { author, content });
                    e.target.reset(); // 폼 초기화
                }
            });

            // 페이지 로드 시 게시글 불러오기
            fetchPosts();
        });
    </script>
</body>
</html>
