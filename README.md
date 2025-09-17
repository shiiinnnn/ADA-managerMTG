<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>株式会社アドライブエージェント マネージャーMTG</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center p-4">
    <div id="app" class="w-full max-w-4xl p-6 bg-white rounded-xl shadow-lg flex flex-col lg:flex-row gap-6">
        <!-- ローダー -->
        <div id="loader" class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-80 rounded-xl" style="display: none;">
            <div class="animate-spin rounded-full h-16 w-16 border-t-2 border-b-2 border-blue-500"></div>
        </div>
        
        <!-- 左パネル: 議題入力 -->
        <div class="lg:w-1/2 p-4 bg-gray-50 rounded-lg shadow-inner">
            <h1 class="text-xl lg:text-2xl font-bold text-gray-800 mb-4 text-center">議題候補の入力</h1>
            <div class="mb-3">
                <label for="meetingDate" class="block text-sm font-medium text-gray-700 mb-1">会議日</label>
                <input type="date" id="meetingDate" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-200">
            </div>
            <div class="mb-3">
                <label for="departmentSelect" class="block text-sm font-medium text-gray-700 mb-1">部署</label>
                <select id="departmentSelect" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-200">
                    <option value="">部署を選択してください</option>
                    <option value="戦略企画部">戦略企画部</option>
                    <option value="CS部">CS部</option>
                    <option value="採用">採用</option>
                    <option value="総務">総務</option>
                    <option value="経理/秘書">経理/秘書</option>
                </select>
            </div>
            <div class="mb-3">
                <label for="nameInput" class="block text-sm font-medium text-gray-700 mb-1">あなたの名前</label>
                <input type="text" id="nameInput" placeholder="例：山田太郎" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-200">
            </div>
            <div class="mb-3">
                <label for="topicInput" class="block text-sm font-medium text-gray-700 mb-1">議題をここに入力してください (複数ある場合は改行)</label>
                <textarea id="topicInput" rows="4" placeholder="例：
新プロジェクトの進捗
CS部からのフィードバック共有
来月のイベント計画" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-200"></textarea>
            </div>
            <button id="submitBtn" class="w-full bg-blue-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-blue-700 transition duration-200 shadow-md">議題を提出</button>
        </div>

        <!-- 右パネル: 議題リストとオーナーコントロール -->
        <div class="lg:w-1/2 p-4 bg-gray-50 rounded-lg shadow-inner">
            <div class="flex items-center justify-between mb-4">
                <h2 id="topicListTitle" class="text-xl lg:text-2xl font-bold text-gray-800">次回の議題候補</h2>
                <button id="toggleOwnerBtn" class="bg-purple-600 text-white font-semibold py-1 px-3 rounded-md text-sm hover:bg-purple-700 transition duration-200 shadow-sm">オーナーモード</button>
            </div>
            <p id="topicListSubtitle" class="text-sm text-gray-600 mb-4">オーナーが会議で取り扱うか選択します。</p>
            
            <button id="toggleArchiveBtn" class="w-full bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-md hover:bg-gray-400 transition duration-200 shadow-sm mb-4">アーカイブを見る</button>
            
            <div id="topicsContainer" class="max-h-96 overflow-y-auto pr-2">
                <ul id="topicList" class="space-y-3">
                    <li id="noTopicsMsg" class="text-center text-gray-500 italic mt-8">まだ議題がありません。</li>
                </ul>
            </div>
            <div id="ownerControls" class="mt-6 pt-4 border-t border-gray-300" style="display: none;">
                <h3 class="text-lg font-semibold text-gray-800 mb-4">オーナー向けコントロール</h3>
                <div class="space-y-3">
                    <button id="clearTopicsBtn" class="w-full bg-red-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-red-700 transition duration-200 shadow-md mb-2">
                        今週の議題候補をクリア
                    </button>
                    <button id="copySelectedBtn" class="w-full bg-green-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-green-700 transition duration-200 shadow-md">
                        選んだ議題をコピー
                    </button>
                    <div id="remindSection" class="p-4 bg-white rounded-md shadow-sm border border-gray-200">
                        <p class="text-sm font-medium text-gray-700 mb-2">リマインダー設定</p>
                        <p id="remindStatus" class="text-sm text-gray-600 italic">自動リマインダーは毎週月曜日 22:00 に実行されます。</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Toast Notification Container -->
    <div id="toast-container" class="fixed top-5 right-5 z-50"></div>

    <!-- Password Modal -->
    <div id="passwordModal" class="fixed inset-0 bg-gray-900 bg-opacity-75 flex items-center justify-center hidden">
        <div class="bg-white p-6 rounded-lg shadow-xl w-full max-w-sm">
            <h4 class="text-lg font-bold text-gray-800 mb-4">オーナーモードのパスワード</h4>
            <input type="password" id="passwordInput" placeholder="パスワードを入力" class="w-full px-4 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
            <div class="mt-4 flex justify-end space-x-2">
                <button id="cancelPasswordBtn" class="bg-gray-300 text-gray-800 font-semibold py-2 px-4 rounded-md hover:bg-gray-400 transition duration-200">キャンセル</button>
                <button id="confirmPasswordBtn" class="bg-purple-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-purple-700 transition duration-200">確定</button>
            </div>
        </div>
    </div>

    <script type="module">
        // Firebase SDKs
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, where, addDoc, getDocs, Timestamp, writeBatch } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Set Firebase debug log level
        setLogLevel('debug');

        // Global variables for Firebase configuration
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // UI Elements
        const appContainer = document.getElementById('app');
        const loader = document.getElementById('loader');
        const meetingDateInput = document.getElementById('meetingDate');
        const topicInput = document.getElementById('topicInput');
        const nameInput = document.getElementById('nameInput');
        const departmentSelect = document.getElementById('departmentSelect');
        const submitBtn = document.getElementById('submitBtn');
        const topicList = document.getElementById('topicList');
        const noTopicsMsg = document.getElementById('noTopicsMsg');
        const topicListTitle = document.getElementById('topicListTitle');
        const topicListSubtitle = document.getElementById('topicListSubtitle');
        const toggleOwnerBtn = document.getElementById('toggleOwnerBtn');
        const ownerControls = document.getElementById('ownerControls');
        const clearTopicsBtn = document.getElementById('clearTopicsBtn');
        const copySelectedBtn = document.getElementById('copySelectedBtn');
        const toastContainer = document.getElementById('toast-container');
        const remindStatus = document.getElementById('remindStatus');
        const passwordModal = document.getElementById('passwordModal');
        const passwordInput = document.getElementById('passwordInput');
        const cancelPasswordBtn = document.getElementById('cancelPasswordBtn');
        const confirmPasswordBtn = document.getElementById('confirmPasswordBtn');
        const toggleArchiveBtn = document.getElementById('toggleArchiveBtn');

        let db, auth;
        let userId = null;
        let isOwner = false;
        let isArchiveView = false;

        // Owner password
        const OWNER_PASSWORD = 'shinkuroS5203ADA';

        function showLoader() {
            loader.style.display = 'flex';
            appContainer.classList.add('pointer-events-none');
        }

        function hideLoader() {
            loader.style.display = 'none';
            appContainer.classList.remove('pointer-events-none');
        }

        function showToast(message, type = 'success') {
            const toast = document.createElement('div');
            toast.className = `p-4 rounded-md shadow-lg mb-3 flex items-center transition-transform duration-300 ease-out transform translate-x-full`;
            
            let bgColor, icon;
            switch (type) {
                case 'success':
                    bgColor = 'bg-green-500';
                    icon = `<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                            </svg>`;
                    break;
                case 'error':
                    bgColor = 'bg-red-500';
                    icon = `<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                            </svg>`;
                    break;
                case 'info':
                default:
                    bgColor = 'bg-blue-500';
                    icon = `<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                            </svg>`;
                    break;
            }

            toast.innerHTML = `<div class="flex-shrink-0 mr-3">${icon}</div><div class="text-white">${message}</div>`;
            toast.classList.add(bgColor);
            toastContainer.appendChild(toast);
            
            setTimeout(() => {
                toast.classList.remove('translate-x-full');
                toast.classList.add('translate-x-0');
            }, 100);

            setTimeout(() => {
                toast.classList.remove('translate-x-0');
                toast.classList.add('translate-x-full');
                toast.addEventListener('transitionend', () => {
                    toast.remove();
                });
            }, 3000);
        }

        async function initializeFirebase() {
            try {
                showLoader();
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                // 認証ロジックを修正
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        setupRealtimeTopics();
                        hideLoader();
                    } else {
                        console.error("ユーザー認証に失敗しました。");
                        hideLoader();
                    }
                });

                const reminderDocRef = doc(db, `artifacts/${appId}/public/data/reminders`, 'weekly-agenda-reminder');
                const reminderDocSnap = await getDoc(reminderDocRef);
                const lastRun = reminderDocSnap.exists() ? reminderDocSnap.data().lastRun.toDate() : null;

                const now = new Date();
                const currentDay = now.getDay();
                const currentHour = now.getHours();

                const isReminderTime = (currentDay === 1 && currentHour >= 22);
                if (isReminderTime && (!lastRun || now.getTime() - lastRun.getTime() > 7 * 24 * 60 * 60 * 1000)) {
                     remindStatus.textContent = "期限です！リマインダーが送信されました。";
                     await setDoc(reminderDocRef, { lastRun: Timestamp.fromDate(now) });
                }
            } catch (error) {
                console.error("Firebaseの初期化に失敗しました:", error);
                hideLoader();
            }
        }

        async function setupRealtimeTopics() {
            if (!userId) {
                console.error("ユーザーIDが利用できません。");
                return;
            }

            const topicsColRef = collection(db, `artifacts/${appId}/public/data/topics`);
            const q = query(topicsColRef);

            onSnapshot(q, (snapshot) => {
                const topics = [];
                snapshot.forEach((doc) => {
                    topics.push({ id: doc.id, ...doc.data() });
                });
                renderTopics(topics);
            }, (error) => {
                console.error("議題のリアルタイム取得に失敗しました:", error);
                showToast("議題の取得中にエラーが発生しました。", "error");
            });
        }
        
        function renderTopics(topics) {
            topicList.innerHTML = '';
            noTopicsMsg.style.display = topics.length === 0 ? 'block' : 'none';
            
            const now = new Date();
            const topicsToDisplay = isArchiveView ? 
                topics.filter(t => t.meetingDate && new Date(t.meetingDate) < now) :
                topics.filter(t => !t.meetingDate || new Date(t.meetingDate) >= now);

            topicsToDisplay.sort((a, b) => {
                const dateA = a.createdAt ? a.createdAt.toDate() : new Date();
                const dateB = b.createdAt ? b.createdAt.toDate() : new Date();
                return dateB - dateA;
            });

            topicsToDisplay.forEach(topic => {
                const li = document.createElement('li');
                li.className = "p-4 bg-white rounded-md shadow-sm border border-gray-200 flex justify-between items-center";
                li.dataset.topicId = topic.id;

                const contentDiv = document.createElement('div');
                contentDiv.className = "flex-grow";
                contentDiv.innerHTML = `
                    <p class="text-sm font-semibold text-gray-900">${topic.topic}</p>
                    <p class="text-xs text-gray-500 mt-1">
                        提出者: ${topic.department} / ${topic.name}
                        (${topic.createdAt.toDate().toLocaleDateString('ja-JP')} ${topic.createdAt.toDate().toLocaleTimeString('ja-JP')})
                    </p>
                    ${topic.meetingDate ? `<p class="text-xs text-gray-500 mt-1">会議日: ${topic.meetingDate}</p>` : ''}
                `;

                li.appendChild(contentDiv);

                if (isOwner) {
                    const controlsDiv = document.createElement('div');
                    controlsDiv.className = "flex items-center space-x-2";
                    
                    if (!isArchiveView) {
                        const checkbox = document.createElement('input');
                        checkbox.type = 'checkbox';
                        checkbox.className = "h-4 w-4 text-blue-600 bg-gray-100 rounded border-gray-300 focus:ring-blue-500";
                        checkbox.dataset.topicText = topic.topic;
                        controlsDiv.appendChild(checkbox);
                    }
                    
                    const statusSelect = document.createElement('select');
                    statusSelect.className = "px-2 py-1 text-sm border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500";
                    
                    const options = [
                        { value: 'unhandled', text: '未処理' },
                        { value: 'handled', text: '取り扱い済み' },
                        { value: 'next_week', text: '次回取扱い予定' }
                    ];

                    options.forEach(option => {
                        const opt = document.createElement('option');
                        opt.value = option.value;
                        opt.textContent = option.text;
                        if (topic.status === option.value) {
                            opt.selected = true;
                        }
                        statusSelect.appendChild(opt);
                    });
                    
                    statusSelect.addEventListener('change', (e) => {
                        updateTopicStatus(topic.id, e.target.value);
                    });

                    controlsDiv.appendChild(statusSelect);
                    li.appendChild(controlsDiv);

                } else {
                    let statusColor = '';
                    let statusText = '';
                    switch (topic.status) {
                        case 'handled':
                            statusColor = 'bg-green-500';
                            statusText = '取り扱い済み';
                            break;
                        case 'next_week':
                            statusColor = 'bg-blue-500';
                            statusText = '次回取扱い予定';
                            break;
                        case 'unhandled':
                        default:
                            statusColor = 'bg-gray-500';
                            statusText = '未処理';
                            break;
                    }
                    const statusBadge = document.createElement('span');
                    statusBadge.className = `${statusColor} text-white text-xs font-semibold px-2 py-1 rounded-full ml-2`;
                    statusBadge.textContent = statusText;
                    li.appendChild(statusBadge);
                }

                topicList.appendChild(li);
            });
        }
        
        async function addTopic() {
            if (!userId) {
                showToast("ユーザー認証が完了していません。しばらくお待ちください。", "error");
                return;
            }
            const meetingDate = meetingDateInput.value;
            const department = departmentSelect.value;
            const name = nameInput.value.trim();
            const topicsText = topicInput.value.trim();

            if (!meetingDate || !department || !name || !topicsText) {
                showToast("全ての項目を入力してください。", "error");
                return;
            }

            showLoader();
            try {
                const topicsArray = topicsText.split('\n').filter(t => t.trim() !== '');
                const topicsColRef = collection(db, `artifacts/${appId}/public/data/topics`);
                
                const batch = writeBatch(db);
                topicsArray.forEach(topic => {
                    const newDocRef = doc(topicsColRef);
                    batch.set(newDocRef, {
                        meetingDate: meetingDate,
                        topic: topic.trim(),
                        department: department,
                        name: name,
                        authorId: userId,
                        createdAt: Timestamp.now(),
                        status: 'unhandled' // Default status
                    });
                });
                
                await batch.commit();

                showToast("議題を提出しました！", "success");

            } catch (error) {
                console.error("議題の追加に失敗しました:", error);
                showToast("議題の提出中にエラーが発生しました。", "error");
            } finally {
                hideLoader();
            }
        }

        async function updateTopicStatus(topicId, status) {
            if (!isOwner) return;
            showLoader();
            try {
                const topicRef = doc(db, `artifacts/${appId}/public/data/topics`, topicId);
                await updateDoc(topicRef, { status: status, lastUpdated: Timestamp.now() });
                showToast("議題のステータスを更新しました。", "success");
            } catch (error) {
                console.error("議題ステータスの更新に失敗しました:", error);
                showToast("議題ステータスの更新中にエラーが発生しました。", "error");
            } finally {
                hideLoader();
            }
        }

        async function clearAllTopics() {
            if (!isOwner) return;
            showLoader();
            try {
                const topicsColRef = collection(db, `artifacts/${appId}/public/data/topics`);
                const snapshot = await getDocs(topicsColRef);
                const batch = writeBatch(db);
                snapshot.docs.forEach((doc) => {
                    batch.delete(doc.ref);
                });
                await batch.commit();
                showToast("すべての議題をクリアしました。", "success");
            } catch (error) {
                console.error("議題のクリアに失敗しました:", error);
                showToast("議題のクリア中にエラーが発生しました。", "error");
            } finally {
                hideLoader();
            }
        }

        function copySelectedTopics() {
            const checkboxes = document.querySelectorAll('#topicList input[type="checkbox"]:checked');
            const selectedTopics = Array.from(checkboxes).map(checkbox => `- ${checkbox.getAttribute('data-topic-text')}`);
            
            if (selectedTopics.length === 0) {
                showToast("コピーする議題を選択してください。", "info");
                return;
            }

            const textToCopy = selectedTopics.join('\n');
            
            const tempTextarea = document.createElement('textarea');
            tempTextarea.value = textToCopy;
            document.body.appendChild(tempTextarea);
            tempTextarea.select();
            
            try {
                const successful = document.execCommand('copy');
                if (successful) {
                    showToast("選んだ議題をクリップボードにコピーしました！", "success");
                } else {
                    showToast("コピーに失敗しました。お使いのブラウザではサポートされていない可能性があります。", "error");
                }
            } catch (err) {
                console.error('Copy command failed:', err);
                showToast("コピーに失敗しました。", "error");
            } finally {
                document.body.removeChild(tempTextarea);
            }
        }

        function toggleOwnerMode() {
            if (isOwner) {
                isOwner = false;
                ownerControls.style.display = 'none';
                toggleOwnerBtn.textContent = 'オーナーモード';
                setupRealtimeTopics();
                showToast("参加者モードに切り替わりました。", "info");
            } else {
                passwordModal.classList.remove('hidden');
                passwordInput.focus();
            }
        }

        function toggleArchiveView() {
            isArchiveView = !isArchiveView;
            if (isArchiveView) {
                topicListTitle.textContent = '議題アーカイブ';
                topicListSubtitle.textContent = '過去の議題履歴です。';
                toggleArchiveBtn.textContent = '次回の議題候補を見る';
            } else {
                topicListTitle.textContent = '次回の議題候補';
                topicListSubtitle.textContent = 'オーナーが会議で取り扱うか選択します。';
                toggleArchiveBtn.textContent = 'アーカイブを見る';
            }
            setupRealtimeTopics();
        }

        function handlePasswordConfirmation() {
            const enteredPassword = passwordInput.value;
            if (enteredPassword === OWNER_PASSWORD) {
                isOwner = true;
                ownerControls.style.display = 'block';
                toggleOwnerBtn.textContent = '参加者モード';
                setupRealtimeTopics();
                showToast("オーナーモードに切り替わりました。", "success");
                passwordModal.classList.add('hidden');
                passwordInput.value = '';
            } else {
                showToast("パスワードが違います。", "error");
                passwordInput.value = '';
            }
        }

        // Event Listeners
        submitBtn.addEventListener('click', addTopic);
        topicInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && e.shiftKey) {
                e.preventDefault();
                addTopic();
            }
        });
        toggleOwnerBtn.addEventListener('click', toggleOwnerMode);
        clearTopicsBtn.addEventListener('click', clearAllTopics);
        copySelectedBtn.addEventListener('click', copySelectedTopics);
        toggleArchiveBtn.addEventListener('click', toggleArchiveView);

        confirmPasswordBtn.addEventListener('click', handlePasswordConfirmation);
        passwordInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                handlePasswordConfirmation();
            }
        });
        cancelPasswordBtn.addEventListener('click', () => {
            passwordModal.classList.add('hidden');
            passwordInput.value = '';
        });


        // Initial setup
        window.onload = initializeFirebase;
    </script>
</body>
</html>
