import React, { useState, useEffect } from 'react';
import { initializeApp } from 'firebase/app';
import { 
    getAuth, 
    onAuthStateChanged,
    createUserWithEmailAndPassword,
    signInWithEmailAndPassword,
    sendPasswordResetEmail,
    signInWithPopup,
    OAuthProvider,
    signOut
} from 'firebase/auth';
import { getFirestore, collection, doc, setDoc, onSnapshot, query, where, updateDoc, serverTimestamp, addDoc } from 'firebase/firestore';
import { Crown, Briefcase, Shield, Calendar, Users, BarChart2, Utensils, LogOut, ChevronLeft, CheckCircle, XCircle, DollarSign, UserCheck, Loader2, Edit, ClipboardCheck, Mail, KeyRound } from 'lucide-react';

// --- Firebase 설정 (Canvas 환경에서 자동으로 제공됨) ---
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
    // 로컬 테스트용 Firebase 설정 예시
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_AUTH_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_STORAGE_BUCKET",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
};
const appId = typeof __app_id !== 'undefined' ? __app_id : 'pt-fit-app-default';

// --- 애플리케이션 메인 컴포넌트 ---
export default function App() {
    const [user, setUser] = useState(null);
    const [authStatus, setAuthStatus] = useState('loading'); // 'loading', 'unauthenticated', 'authenticated'
    const [view, setView] = useState('dashboard');
    
    const [db, setDb] = useState(null);
    const [auth, setAuth] = useState(null);

    useEffect(() => {
        try {
            const app = initializeApp(firebaseConfig);
            const authInstance = getAuth(app);
            const dbInstance = getFirestore(app);
            setAuth(authInstance); setDb(dbInstance);

            const unsubscribe = onAuthStateChanged(authInstance, (firebaseUser) => {
                if (firebaseUser) {
                    const userDocRef = doc(dbInstance, "users", firebaseUser.uid);
                    const unsubDoc = onSnapshot(userDocRef, (docSnap) => {
                        if (docSnap.exists()) {
                            const userData = docSnap.data();
                            if (userData.status === 'approved') {
                                setUser({ uid: firebaseUser.uid, ...userData });
                                setAuthStatus('authenticated');
                            } else {
                                authInstance.signOut(); // 비승인/비활성 사용자는 로그아웃
                                alert(`계정이 비활성화되었거나 승인 대기 중입니다. 관리자에게 문의하세요.`);
                            }
                        } else {
                           setAuthStatus('unauthenticated');
                        }
                    });
                    return () => unsubDoc();
                } else {
                    setUser(null);
                    setAuthStatus('unauthenticated');
                }
            });
            return () => unsubscribe();
        } catch (error) { console.error("Firebase 초기화 오류:", error); setAuthStatus('unauthenticated'); }
    }, []);

    const getRoleName = (r) => ({ owner: '오너', president: '대표', manager: '매니저', trainer: '트레이너', member: '회원' }[r] || '사용자');
    
    if (authStatus === 'loading') {
        return <div className="flex items-center justify-center h-screen bg-gray-900 text-white"><Loader2 className="animate-spin mr-2" /><span>로딩 중...</span></div>;
    }

    if (authStatus === 'unauthenticated') {
        return <LoginScreen auth={auth} db={db} />;
    }

    return (
        <div className="flex h-screen bg-gray-900 text-white font-sans">
            <Sidebar role={user.role} setView={setView} onLogout={() => signOut(auth)} getRoleName={getRoleName} />
            <main className="flex-1 p-4 sm:p-6 lg:p-8 overflow-y-auto">
                <CurrentView view={view} role={user.role} db={db} user={user} />
            </main>
        </div>
    );
}

// --- 로그인/회원가입 컴포넌트 ---
function LoginScreen({ auth, db }) {
    const [page, setPage] = useState('login'); // 'login', 'signup', 'reset'
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const [loading, setLoading] = useState(false);
    const [message, setMessage] = useState('');

    const handleEmailLogin = async (e) => {
        e.preventDefault();
        setLoading(true); setError('');
        try {
            await signInWithEmailAndPassword(auth, email, password);
        } catch (err) {
            setError('이메일 또는 비밀번호가 올바르지 않습니다.');
        } finally {
            setLoading(false);
        }
    };

    const handleSignUp = async (e) => {
        e.preventDefault();
        setLoading(true); setError('');
        try {
            const userCredential = await createUserWithEmailAndPassword(auth, email, password);
            const user = userCredential.user;
            // 기본 역할은 '회원'으로, 승인 상태로 시작
            await setDoc(doc(db, "users", user.uid), {
                email: user.email,
                name: user.email.split('@')[0], // 초기 이름
                role: 'member',
                status: 'approved',
                createdAt: serverTimestamp(),
            });
            setMessage('회원가입이 완료되었습니다. 로그인해주세요.');
            setPage('login');
        } catch (err) {
            if (err.code === 'auth/email-already-in-use') {
                setError('이미 사용 중인 이메일입니다.');
            } else {
                setError('회원가입에 실패했습니다.');
            }
        } finally {
            setLoading(false);
        }
    };
    
    const handlePasswordReset = async (e) => {
        e.preventDefault();
        setLoading(true); setError(''); setMessage('');
        try {
            await sendPasswordResetEmail(auth, email);
            setMessage('비밀번호 재설정 이메일을 보냈습니다. 받은편지함을 확인해주세요.');
        } catch (err) {
            setError('이메일 발송에 실패했습니다. 이메일 주소를 확인해주세요.');
        } finally {
            setLoading(false);
        }
    };

    const handleKakaoLogin = async () => {
        // 실제 서비스에서는 Firebase 콘솔에서 카카오 로그인을 활성화하고
        // 클라이언트 ID와 시크릿 키를 설정해야 합니다.
        const provider = new OAuthProvider('kakao.com');
        try {
            const result = await signInWithPopup(auth, provider);
            const user = result.user;
            // DB에 카카오 유저 정보 저장 또는 업데이트
            await setDoc(doc(db, "users", user.uid), {
                email: user.email,
                name: user.displayName,
                photoURL: user.photoURL,
                role: 'member', // 신규 유저는 기본적으로 회원
                status: 'approved'
            }, { merge: true });
        } catch (err) {
            setError('카카오 로그인에 실패했습니다.');
            console.error(err);
        }
    };

    const renderForm = () => {
        switch (page) {
            case 'signup':
                return (
                    <form onSubmit={handleSignUp} className="space-y-6">
                        <h2 className="text-2xl font-bold text-center">신규 등록</h2>
                        <div><input type="email" placeholder="이메일" value={email} onChange={(e)=>setEmail(e.target.value)} required className="w-full p-3 bg-gray-700 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" /></div>
                        <div><input type="password" placeholder="비밀번호 (6자 이상)" value={password} onChange={(e)=>setPassword(e.target.value)} required className="w-full p-3 bg-gray-700 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" /></div>
                        <button type="submit" disabled={loading} className="w-full p-3 bg-blue-600 hover:bg-blue-700 rounded-md font-bold transition flex justify-center">{loading ? <Loader2 className="animate-spin"/> : '등록하기'}</button>
                        <p className="text-center"><button type="button" onClick={() => setPage('login')} className="text-blue-400 hover:underline">로그인 화면으로</button></p>
                    </form>
                );
            case 'reset':
                 return (
                    <form onSubmit={handlePasswordReset} className="space-y-6">
                        <h2 className="text-2xl font-bold text-center">비밀번호 찾기</h2>
                        <div><input type="email" placeholder="가입한 이메일 주소" value={email} onChange={(e)=>setEmail(e.target.value)} required className="w-full p-3 bg-gray-700 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" /></div>
                        <button type="submit" disabled={loading} className="w-full p-3 bg-blue-600 hover:bg-blue-700 rounded-md font-bold transition flex justify-center">{loading ? <Loader2 className="animate-spin"/> : '재설정 이메일 받기'}</button>
                        <p className="text-center"><button type="button" onClick={() => setPage('login')} className="text-blue-400 hover:underline">로그인 화면으로</button></p>
                    </form>
                );
            default: // login
                return (
                    <form onSubmit={handleEmailLogin} className="space-y-6">
                        <div className="relative"><Mail className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400" size={20} /><input type="email" placeholder="이메일" value={email} onChange={(e)=>setEmail(e.target.value)} required className="w-full p-3 pl-10 bg-gray-700 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" /></div>
                        <div className="relative"><KeyRound className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400" size={20} /><input type="password" placeholder="비밀번호" value={password} onChange={(e)=>setPassword(e.target.value)} required className="w-full p-3 pl-10 bg-gray-700 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" /></div>
                        <button type="submit" disabled={loading} className="w-full p-3 bg-blue-600 hover:bg-blue-700 rounded-md font-bold transition flex justify-center">{loading ? <Loader2 className="animate-spin"/> : '로그인'}</button>
                        <div className="flex justify-between text-sm">
                            <button type="button" onClick={() => setPage('signup')} className="text-gray-400 hover:text-white">신규 등록</button>
                            <button type="button" onClick={() => setPage('reset')} className="text-gray-400 hover:text-white">비밀번호 찾기</button>
                        </div>
                        <div className="relative flex py-2 items-center"><div className="flex-grow border-t border-gray-600"></div><span className="flex-shrink mx-4 text-gray-500 text-sm">또는</span><div className="flex-grow border-t border-gray-600"></div></div>
                        <button type="button" onClick={handleKakaoLogin} className="w-full p-3 bg-yellow-400 text-black font-bold rounded-md flex items-center justify-center gap-2 hover:bg-yellow-500 transition">
                            <svg className="w-5 h-5" viewBox="0 0 32 32"><path fill="currentColor" d="M16 4.64c-6.96 0-12.64 4.48-12.64 10.08 0 3.52 2.32 6.56 5.76 8.48l-1.92 7.04 7.68-4.16c.4.08.72.08 1.12.08 6.96 0 12.64-4.48 12.64-10.08S22.96 4.64 16 4.64z"></path></svg>
                            카카오톡으로 시작하기
                        </button>
                    </form>
                );
        }
    };
    
    return (
        <div className="min-h-screen bg-gray-900 flex flex-col items-center justify-center p-4" style={{backgroundImage: 'linear-gradient(rgba(17, 24, 39, 0.8), rgba(17, 24, 39, 1)), url(https://placehold.co/1920x1080/111827/1f2937?text=.)', backgroundSize:'cover', backgroundPosition: 'center' }}>
            <div className="w-full max-w-md">
                <div className="text-center mb-8">
                    <h1 className="text-4xl font-bold text-blue-400 mb-2">PT-Fit</h1>
                    <p className="text-gray-300">당신의 피트니스 비즈니스를 위한 모든 것</p>
                </div>
                <div className="bg-gray-800/80 backdrop-blur-sm p-8 rounded-2xl shadow-2xl">
                    {message && <p className="mb-4 text-center text-green-400">{message}</p>}
                    {error && <p className="mb-4 text-center text-red-400">{error}</p>}
                    {renderForm()}
                </div>
            </div>
        </div>
    );
}


// --- 나머지 컴포넌트 ---
// Sidebar, CurrentView, 승인 컴포넌트 등은 이전 버전의 코드를 기반으로 유지됩니다.
// 단, 로그인 방식 변경에 따라 불필요해진 onLogin, getRoleName 등의 prop은 제거되었습니다.

function Sidebar({ role, setView, onLogout, getRoleName }) {
    const navItems = {
        owner: [ { icon: UserCheck, label: '계정 승인 및 관리', view: 'owner-approval' }, { icon: Building, label: '지점별 성과', view: 'branch-performance' } ],
        president: [ { icon: Shield, label: '매니저 승인', view: 'president-approval' }, { icon: BarChart2, label: '담당 지점 성과', view: 'branch-performance' } ],
        manager: [ { icon: BarChart2, label: '지점 매출 관리', view: 'sales' }, { icon: Calendar, label: '스케줄 관리', view: 'calendar' }, { icon: ClipboardCheck, label: '수정 요청 승인', view: 'manager-edit-approval' } ],
        trainer: [ { icon: Calendar, label: '내 스케줄', view: 'calendar' }, { icon: DollarSign, label: '내 매출', view: 'my-sales' } ],
        member: [ { icon: Calendar, label: '내 수업 일정', view: 'calendar' }, { icon: Utensils, label: '내 식단', view: 'diet' } ],
    };

    return (
        <nav className="w-64 bg-gray-800 p-5 flex flex-col justify-between shadow-lg">
            <div>
                <div className="mb-10"><h1 className="text-2xl font-bold text-blue-400">PT-Fit</h1><span className="text-sm text-gray-400">{getRoleName(role)} 모드</span></div>
                <ul>{navItems[role]?.map((item, index) => (<li key={index} className="mb-4"><button onClick={() => setView(item.view)} className="flex items-center w-full text-lg text-gray-300 hover:text-white hover:bg-gray-700 p-3 rounded-lg transition duration-200"><item.icon className="mr-4" />{item.label}</button></li>))}</ul>
            </div>
            <button onClick={onLogout} className="flex items-center w-full text-lg text-gray-300 hover:text-white hover:bg-red-600 p-3 rounded-lg transition duration-200"><LogOut className="mr-4"/>로그아웃</button>
        </nav>
    );
}

function CurrentView({ view, role, db, user }) {
    const viewProps = { role, db, user };
    const views = {
        'owner-approval': OwnerApprovalView,
        'president-approval': PresidentApprovalView,
        'manager-edit-approval': ManagerEditApprovalView,
        'my-sales': TrainerSalesView,
    };
    const Component = views[view] || (() => <div className="bg-gray-800 p-6 rounded-lg text-white"><h2>대시보드</h2><p>"{view}" 화면 컨텐츠가 여기에 표시됩니다.</p></div>);
    return <Component {...viewProps} />;
}

function OwnerApprovalView({ db }) {
    const [pendingUsers, setPendingUsers] = useState([]);
    const [approvedPresidents, setApprovedPresidents] = useState([]);

    useEffect(() => {
        const q_pending = query(collection(db, 'users'), where('status', '==', 'pending'));
        const unsub_pending = onSnapshot(q_pending, snapshot => setPendingUsers(snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }))));

        const q_approved = query(collection(db, 'users'), where('role', '==', 'president'));
        const unsub_approved = onSnapshot(q_approved, snapshot => setApprovedPresidents(snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }))));

        return () => { unsub_pending(); unsub_approved(); };
    }, [db]);
    
    const handleApprove = async (userId, newStatus) => {
        await updateDoc(doc(db, 'users', userId), { status: newStatus });
    };

    const handleToggleAccess = async (userId, currentStatus) => {
        await updateDoc(doc(db, 'users', userId), { status: currentStatus === 'approved' ? 'disabled' : 'approved' });
    }

    return (
        <div>
            <h2 className="text-3xl font-bold mb-6 text-yellow-400">계정 승인 및 관리</h2>
            <div className="bg-gray-800 p-6 rounded-lg mb-6">
                <h3 className="text-xl font-semibold mb-4 text-gray-300">신규 계정 승인 요청 ({pendingUsers.length}건)</h3>
                <div className="space-y-4">
                    {pendingUsers.length > 0 ? pendingUsers.map(p => (<div key={p.id} className="p-4 bg-gray-700 rounded-lg flex justify-between items-center"><span>{p.name} ({p.role})</span><div className="flex gap-2"><button onClick={() => handleApprove(p.id, 'approved')} className="bg-green-600 hover:bg-green-700 font-bold py-2 px-4 rounded-md">승인</button><button onClick={() => handleApprove(p.id, 'denied')} className="bg-red-600 hover:bg-red-700 font-bold py-2 px-4 rounded-md">거절</button></div></div>)) : <p className="text-gray-500">새로운 승인 요청이 없습니다.</p>}
                </div>
            </div>
            <div className="bg-gray-800 p-6 rounded-lg">
                <h3 className="text-xl font-semibold mb-4 text-gray-300">대표(President) 계정 접근 관리</h3>
                 <div className="space-y-4">
                     {approvedPresidents.map(p => (<div key={p.id} className="p-4 bg-gray-700 rounded-lg flex justify-between items-center"><span>{p.name}</span><div className="flex items-center gap-3"><span className={`text-sm font-bold ${p.status === 'approved' ? 'text-green-400' : 'text-red-400'}`}>{p.status === 'approved' ? '활성화' : '비활성화'}</span><button onClick={() => handleToggleAccess(p.id, p.status)} className="bg-indigo-600 hover:bg-indigo-700 font-bold py-2 px-4 rounded-md">{p.status === 'approved' ? '비활성화' : '활성화'}</button></div></div>))}
                 </div>
            </div>
        </div>
    );
}

function PresidentApprovalView({ db, user }) { return (<div><h2 className="text-3xl font-bold mb-6 text-indigo-400">매니저 승인</h2><div className="bg-gray-800 p-6 rounded-lg">대표가 관리하는 지점의 매니저 승인 요청 목록이 여기에 표시됩니다.</div></div>) }
function ManagerEditApprovalView({ db, user }) { return (<div><h2 className="text-3xl font-bold mb-6 text-blue-400">트레이너 수정 요청 승인</h2><div className="bg-gray-800 p-6 rounded-lg">매니저가 관리하는 트레이너의 수정 요청 목록이 여기에 표시됩니다.</div></div>) }
function TrainerSalesView({ db, user }) {
    const handleRequestEdit = async () => {
        await addDoc(collection(db, 'editRequests'), {
            trainerId: user.uid, trainerName: user.name, type: 'sales',
            from: '6월 매출: 3,000,000원', to: '6월 매출: 3,500,000원',
            status: 'pending', createdAt: serverTimestamp(),
        });
        alert('매출 기록 수정 요청이 매니저에게 전송되었습니다.');
    };
    return (<div><h2 className="text-3xl font-bold mb-6 text-green-400">내 매출</h2><div className="bg-gray-800 p-6 rounded-lg"><p className="mb-4">나의 월별 매출 기록이 여기에 표시됩니다.</p><button onClick={handleRequestEdit} className="bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-2 px-4 rounded-lg flex items-center gap-2"><Edit size={16} /> 수정 요청하기</button></div></div>);
}
