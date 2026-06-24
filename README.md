import React, { useState, useEffect, useRef } from "react";
import { motion, AnimatePresence } from "motion/react";
import {
  Sword,
  Shield,
  Map as MapIcon,
  Users,
  Coins,
  Crown,
  ScrollText,
  Tent,
  Sparkles,
  MessageSquare,
  Package,
  BookOpen,
  Flame,
  Flag,
  Activity,
  Send,
  Castle,
  Store,
  ArrowUpCircle,
  Save,
  ZoomIn,
  ZoomOut,
  AlertTriangle,
  Megaphone,
  X,
  Ship,
  Anchor,
  Trophy,
  Handshake,
  Skull,
  RefreshCw,
  Star,
} from "lucide-react";
import {
  doc,
  setDoc,
  getDoc,
  serverTimestamp,
  collection,
  onSnapshot,
  updateDoc,
  deleteDoc,
  query,
  where,
  orderBy,
} from "firebase/firestore";
import {
  signInAnonymously,
  onAuthStateChanged,
  User,
  signInWithPopup,
} from "firebase/auth";

import {
  auth,
  db,
  handleFirestoreError,
  OperationType,
  testConnection,
  googleProvider,
} from "./lib/firebase";
import { soundManager } from "./lib/sound";
import { Volume2, VolumeX } from "lucide-react";
import {
  TROOP_TYPES,
  FACTIONS,
  FACTION_COLORS,
  PLAYER_COLOR,
  MASTER_GENERALS,
  MASTER_WEAPONS,
  INITIAL_LANDS,
  ROUTE_CONNECTIONS,
} from "./constants";

const generateId = () => Math.random().toString(36).substring(2, 11);
const generate7DigitCode = () => Math.floor(1000000 + Math.random() * 9000000).toString();
const APP_ID = "804f5516-8d36-4ca5-a022-bd8dcd05d0ea";

// --- Components ---

const FactionKamon = ({
  faction,
  className = "w-10 h-10",
}: {
  faction: string;
  className?: string;
}): React.ReactElement => {
  const kamons: Record<string, React.ReactElement> = {
    織田: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <circle
          cx="50"
          cy="50"
          r="40"
          fill="none"
          stroke="currentColor"
          strokeWidth="6"
        />
        <circle
          cx="50"
          cy="50"
          r="15"
          fill="none"
          stroke="currentColor"
          strokeWidth="6"
        />
        <rect x="25" y="47" width="50" height="6" fill="currentColor" />
        <rect x="47" y="25" width="6" height="50" fill="currentColor" />
      </svg>
    ),
    武田: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <polygon points="50,15 65,30 50,45 35,30" />
        <polygon points="50,55 65,70 50,85 35,70" />
        <polygon points="25,35 40,50 25,65 10,50" />
        <polygon points="75,35 90,50 75,65 60,50" />
      </svg>
    ),
    島津: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <circle
          cx="50"
          cy="50"
          r="45"
          fill="none"
          stroke="currentColor"
          strokeWidth="10"
        />
        <rect x="20" y="42" width="60" height="16" fill="currentColor" />
        <rect x="42" y="20" width="16" height="60" fill="currentColor" />
      </svg>
    ),
    北条: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <polygon points="50,15 70,50 30,50" />
        <polygon points="25,60 45,95 5,95" />
        <polygon points="75,60 95,95 55,95" />
      </svg>
    ),
    毛利: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <rect x="20" y="25" width="60" height="15" fill="currentColor" />
        <circle cx="25" cy="65" r="15" />
        <circle cx="50" cy="65" r="15" />
        <circle cx="75" cy="65" r="15" />
      </svg>
    ),
    徳川: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <circle
          cx="50"
          cy="50"
          r="45"
          fill="none"
          stroke="currentColor"
          strokeWidth="5"
        />
        <path d="M50 50 C 30 10 70 10 50 50 Z" />
        <path d="M50 50 C 10 40 20 80 50 50 Z" />
        <path d="M50 50 C 90 40 80 80 50 50 Z" />
      </svg>
    ),
    伊達: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <circle
          cx="50"
          cy="50"
          r="45"
          fill="none"
          stroke="currentColor"
          strokeWidth="5"
        />
        <circle
          cx="50"
          cy="50"
          r="30"
          fill="none"
          stroke="currentColor"
          strokeWidth="3"
        />
        <circle cx="50" cy="50" r="15" fill="currentColor" />
        <circle cx="30" cy="30" r="5" fill="currentColor" />
        <circle cx="70" cy="30" r="5" fill="currentColor" />
        <circle cx="30" cy="70" r="5" fill="currentColor" />
        <circle cx="70" cy="70" r="5" fill="currentColor" />
        <circle cx="50" cy="20" r="5" fill="currentColor" />
        <circle cx="50" cy="80" r="5" fill="currentColor" />
        <circle cx="20" cy="50" r="5" fill="currentColor" />
        <circle cx="80" cy="50" r="5" fill="currentColor" />
      </svg>
    ),
    上杉: (
      <svg viewBox="0 0 100 100" className={className} fill="currentColor">
        <circle
          cx="50"
          cy="50"
          r="45"
          fill="none"
          stroke="currentColor"
          strokeWidth="5"
        />
        <polygon
          points="50,20 70,80 30,80"
          fill="none"
          stroke="currentColor"
          strokeWidth="5"
        />
        <circle cx="50" cy="55" r="15" fill="currentColor" />
      </svg>
    ),
  };

  if (kamons[faction]) return kamons[faction];
  return (
    <svg viewBox="0 0 100 100" className={className} fill="currentColor">
      <circle
        cx="50"
        cy="50"
        r="45"
        fill="none"
        stroke="currentColor"
        strokeWidth="5"
      />
      <text
        x="50"
        y="66"
        fontSize="45"
        textAnchor="middle"
        fill="currentColor"
        fontWeight="bold"
        fontFamily="serif"
      >
        {faction ? faction[0] : "群"}
      </text>
    </svg>
  );
};

const RarityStars = ({ count }: { count: number }) => {
  const colors = [
    "text-stone-600", // 1
    "text-stone-400", // 2
    "text-orange-600", // 3 (Bronze)
    "text-slate-300", // 4 (Silver)
    "text-yellow-400", // 5 (Gold)
  ];
  const activeColor = colors[count - 1] || "text-yellow-400";

  return (
    <div className="flex gap-0.5">
      {[...Array(count)].map((_, i) => (
        <Star key={i} className={`w-2.5 h-2.5 ${activeColor} fill-current`} />
      ))}
    </div>
  );
};

// --- Main App ---

export default function App() {
  const [user, setUser] = useState<User | null>(null);
  const [authError, setAuthError] = useState<string | null>(null);
  const [isMuted, setIsMuted] = useState(soundManager.getMutedState());
  const [activeTab, setActiveTab] = useState("map");
  const [selectedNavyPortId, setSelectedNavyPortId] = useState<number | null>(1);
  const [showCastleList, setShowCastleList] = useState(true);
  const [gold, setGold] = useState(20000);
  const [copper, setCopper] = useState(25000);
  const [hasNavy, setHasNavy] = useState(false);
  const [navyShips, setNavyShips] = useState<Record<string, number>>({
    sengokubune: 0,
    kobaya: 0,
    sekibune: 0,
    atakebune: 0,
  });
  const [diplomacyRelations, setDiplomacyRelations] = useState<
    Record<string, string>
  >({});
  const [peoplesLoyalty, setPeoplesLoyalty] = useState(100);
  const [capitalLandId, setCapitalLandId] = useState<number | null>(1);
  const [lands, setLands] = useState(INITIAL_LANDS);
  const [inventory, setInventory] = useState<Record<string, number>>({
    tactics: 5,
    drum: 3,
    hora: 2,
  });
  const [activeBuff, setActiveBuff] = useState<string | null>(null);
  const [showItemModal, setShowItemModal] = useState<string | null>(null);
  const [inventoryWeapons, setInventoryWeapons] = useState<
    Record<string, number>
  >({ wp1: 2, wp2: 1, wp3: 1 });
  const [equippedWeapons, setEquippedWeapons] = useState<
    Record<string, string>
  >({});
  const [showWeaponModalFor, setShowWeaponModalFor] = useState<string | null>(
    null,
  );
  const [shopBuyQuantity, setShopBuyQuantity] = useState<number>(1);
  const [shopQuantities, setShopQuantities] = useState<Record<string, number>>({});
  const [trainingGeneral, setTrainingGeneral] = useState<any | null>(null);
  const [merchantEvent, setMerchantEvent] = useState<any>(null);
  const [filterFaction, setFilterFaction] = useState("ALL");
  const [ownedGenerals, setOwnedGenerals] = useState<any[]>([]);
  const [decks, setDecks] = useState<(string | null)[][]>([
    [null, null, null, null, null],
    [null, null, null, null, null],
    [null, null, null, null, null],
  ]);
  const [activeDeckIndex, setActiveDeckIndex] = useState(0);
  const [selectedDeckForBattle, setSelectedDeckForBattle] = useState(0);
  const [occupiedLands, setOccupiedLands] = useState<number[]>([]);
  const [items, setItems] = useState<any[]>([]);
  const [generalFilter, setGeneralFilter] = useState("ALL");
  const [generalSort, setGeneralSort] = useState("RARITY");
  const [battlingLandId, setBattlingLandId] = useState<number | null>(null);
  const [selectedLand, setSelectedLand] = useState<any>(
    INITIAL_LANDS.find((l) => l.id === 1) || null,
  );
  const [battleScene, setBattleScene] = useState<any>(null);

  // --- Online PvP State ---
  const [pvpRoomsList, setPvpRoomsList] = useState<any[]>([]);
  const [pvpActiveRoom, setPvpActiveRoom] = useState<any | null>(null);
  const [pvpMatchmaking, setPvpMatchmaking] = useState(false);
  const [pvpRoomNameInput, setPvpRoomNameInput] = useState("");
  const [pvpJoinCodeInput, setPvpJoinCodeInput] = useState("");
  const [pvpSelectedDeckIdx, setPvpSelectedDeckIdx] = useState(0);
  const [pvpCustomName, setPvpCustomName] = useState("");
  const [pvpRewardedRooms, setPvpRewardedRooms] = useState<Record<string, boolean>>({});
  const pvpUnsubscribeRef = useRef<(() => void) | null>(null);

  const [gachaResult, setGachaResult] = useState<any[] | null>(null);
  const [isGameClear, setIsGameClear] = useState(false);
  const [isGameOver, setIsGameOver] = useState(false);
  const [showResetModal, setShowResetModal] = useState(false);
  const [castleFilter, setCastleFilter] = useState<
    "all" | "mine" | "attackable" | "others"
  >("all");
  const [chatMessages, setChatMessages] = useState<any[]>([
    {
      id: "m1",
      sender: "柴田権六",
      text: "御館様、よろしくお頼み申す！",
      isSelf: false,
    },
    {
      id: "m2",
      sender: "木下藤吉郎",
      text: "ははっ！微力ながら尽力いたしまする！",
      isSelf: false,
    },
  ]);
  const [chatInput, setChatInput] = useState("");
  const [taxCooldown, setTaxCooldown] = useState(0);
  const [ikkiEvent, setIkkiEvent] = useState<any>(null);
  const [invasionEvent, setInvasionEvent] = useState<any>(null);
  const [allianceInfo, setAllianceInfo] = useState({
    name: "天下布武の会",
    level: 3,
    exp: 1500,
    maxExp: 5000,
    members: [
      { name: "御館様 (あなた)", power: 0, role: "盟主" },
      { name: "柴田権六", power: 10500, role: "副盟主" },
      { name: "木下藤吉郎", power: 8900, role: "構成員" },
    ],
    logs: [
      "木下藤吉郎が同盟に加入しました。",
      "柴田権六が1000銅銭を寄付しました。",
    ],
  });
  const [logs, setLogs] = useState<any[]>([
    {
      id: "init",
      text: "いざ出陣じゃ！まずは上の「出陣」タブから地図上の土地を選び、出陣ボタンを押すのだ。",
      type: "system",
    },
  ]);
  const [advisorQuery, setAdvisorQuery] = useState("");
  const [advisorResponse, setAdvisorResponse] = useState("");
  const [isConsulting, setIsConsulting] = useState(false);
  const [isSaving, setIsSaving] = useState(false);
  const [mapScale, setMapScale] = useState(1.0);
  const [isDragging, setIsDragging] = useState(false);
  const [dragStart, setDragStart] = useState({
    x: 0,
    y: 0,
    scrollLeft: 0,
    scrollTop: 0,
  });
  const [dragDistance, setDragDistance] = useState(0);

  const mapContainerRef = useRef<HTMLDivElement>(null);
  const logsEndRef = useRef<HTMLDivElement>(null);
  const chatEndRef = useRef<HTMLDivElement>(null);

  // 薩摩(id: 1, x: 21, y: 85)への初期スクロール処理
  const scrollToSatsuma = () => {
    if (!mapContainerRef.current) return;
    const container = mapContainerRef.current;
    const scrollWidth = 12800 * mapScale;
    const scrollHeight = 9600 * mapScale;
    const targetX = (21 / 100) * scrollWidth - container.clientWidth / 2;
    const targetY = (85 / 100) * scrollHeight - container.clientHeight / 2;
    container.scrollLeft = targetX;
    container.scrollTop = targetY;
  };

  useEffect(() => {
    if (activeTab === "map") {
      const timers = [100, 300, 600, 1000].map((delay) =>
        setTimeout(() => {
          scrollToSatsuma();
        }, delay),
      );
      return () => timers.forEach((t) => clearTimeout(t));
    }
  }, [activeTab, mapScale, user]);

  // Handle Sengoku BGM playing and scene transitions automatically
  useEffect(() => {
    const handleFirstInteraction = () => {
      if (!isMuted) {
        if (battleScene) {
          soundManager.startBgm("combat");
        } else {
          soundManager.startBgm("peace");
        }
      }
      window.removeEventListener("click", handleFirstInteraction);
      window.removeEventListener("touchstart", handleFirstInteraction);
    };
    window.addEventListener("click", handleFirstInteraction);
    window.addEventListener("touchstart", handleFirstInteraction);

    if (isMuted) {
      soundManager.stopBgm();
    } else {
      if (battleScene) {
        soundManager.startBgm("combat");
      } else {
        soundManager.startBgm("peace");
      }
    }

    return () => {
      window.removeEventListener("click", handleFirstInteraction);
      window.removeEventListener("touchstart", handleFirstInteraction);
    };
  }, [battleScene, isMuted]);

  // --- API Handlers ---

  const callAI = async (
    endpoint: string,
    prompt: string,
    systemInstruction?: string,
  ) => {
    try {
      const response = await fetch(`/api/${endpoint}`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ prompt, systemInstruction }),
      });
      if (!response.ok) throw new Error("API request failed");
      const data = await response.json();
      return data.text;
    } catch (error) {
      console.error("AI error:", error);
      return "通信に失敗いたしました。";
    }
  };

  const addLog = (text: string, type = "normal") => {
    setLogs((prev) => [...prev, { id: generateId(), text, type }]);
  };

  // --- Auth & Data Loading ---

  const handleGoogleSignIn = async () => {
    try {
      await signInWithPopup(auth, googleProvider);
      setAuthError(null);
    } catch (err: any) {
      console.error("Google sign-in failed", err);
      setAuthError(`ログイン失敗: ${err.message}`);
    }
  };

  useEffect(() => {
    if (authError) {
      addLog(`${authError}`, "error");
    }
  }, [authError]);

  useEffect(() => {
    // Try anonymous sign-in as background attempt
    signInAnonymously(auth).catch((err) => {
      console.warn("Anonymous auth failed (expected if disabled)", err);
      if (err.code === "auth/admin-restricted-operation") {
        setAuthError(
          "匿名ログイン制限中。Googleアカウントでログインしてください。",
        );
      } else {
        setAuthError(`接続エラー: ${err.message}`);
      }
    });

    const unsubscribe = onAuthStateChanged(auth, (u) => {
      setUser(u);
      if (u) {
        setAuthError(null);
        loadData(u.uid);
      }
    });

    testConnection();
    return () => unsubscribe();
  }, []);

  // --- Real-time PvP Lobby Watcher ---
  useEffect(() => {
    if (activeTab !== "pvp" || !user) {
      setPvpRoomsList([]);
      return;
    }

    const collRef = collection(db, "artifacts", APP_ID, "pvpRooms");
    const q = query(collRef, orderBy("updatedAt", "desc"));
    
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const list: any[] = [];
      snapshot.forEach((doc) => {
        list.push({ ...doc.data(), id: doc.id });
      });
      setPvpRoomsList(list);
    }, (err) => {
      console.error("Error loading PvP rooms:", err);
    });

    return () => unsubscribe();
  }, [activeTab, user]);

  // --- PvP Game Loop & Automatic Resolution / Reward Watcher ---
  useEffect(() => {
    if (!pvpActiveRoom || !user) return;

    // A: Host-only Tactic Resolution
    if (pvpActiveRoom.hostUid === user.uid && pvpActiveRoom.status === "ready") {
      if (pvpActiveRoom.hostTactic && pvpActiveRoom.guestTactic) {
        const timer = setTimeout(() => {
          resolvePvpRound(pvpActiveRoom);
        }, 1200);
        return () => clearTimeout(timer);
      }
    }

    // B: Auto-advance clashed states if both tactics are reset
    if (pvpActiveRoom.hostUid === user.uid && pvpActiveRoom.status === "clashed") {
      if (pvpActiveRoom.hostTactic === "" && pvpActiveRoom.guestTactic === "") {
        const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", pvpActiveRoom.roomId);
        setDoc(docRef, {
          ...pvpActiveRoom,
          status: "ready",
          turn: pvpActiveRoom.turn + 1,
          updatedAt: serverTimestamp(),
        }).catch((err) => console.error("Error advancing turn:", err));
      }
    }

    // C: Award system
    if (pvpActiveRoom.status === "finished" && !pvpRewardedRooms[pvpActiveRoom.roomId]) {
      const isWinner = pvpActiveRoom.winnerUid === user.uid;
      const isTie = !pvpActiveRoom.winnerUid;
      
      let rewardGold = 100;
      let rewardCopper = 1000;
      let rewardMsg = "";

      if (isTie) {
        rewardGold = 200;
        rewardCopper = 1500;
        rewardMsg = `【引き分け】両雄譲らず対峙し引き分け！ 奨励金として小判${rewardGold}、銅銭${rewardCopper.toLocaleString()}を拝領しました。`;
      } else if (isWinner) {
        rewardGold = 500;
        rewardCopper = 3500;
        rewardMsg = `【大勝利！】見事に敵将を討ち取りました！ 恩賞として特別功詞小判${rewardGold}、銅銭${rewardCopper.toLocaleString()}を拝領しました！`;
      } else {
        rewardMsg = `【敗北】敗北の辛酸を舐めました。 再鍛錬を期するための慰労金として小判${rewardGold}、銅銭${rewardCopper.toLocaleString()}を拝領しました。`;
      }

      setGold(prev => prev + rewardGold);
      setCopper(prev => prev + rewardCopper);
      addLog(rewardMsg, isWinner ? "success" : isTie ? "system" : "error");

      setPvpRewardedRooms(prev => ({
        ...prev,
        [pvpActiveRoom.roomId]: true
      }));
    }
  }, [pvpActiveRoom, user]);

  // --- PvP Core Functions ---
  const subscribeToRoom = (roomId: string) => {
    if (pvpUnsubscribeRef.current) {
      pvpUnsubscribeRef.current();
    }
    const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", roomId);
    pvpUnsubscribeRef.current = onSnapshot(docRef, (snapshot) => {
      if (snapshot.exists()) {
        const data = snapshot.data();
        setPvpActiveRoom({ ...data, id: snapshot.id });
      } else {
        setPvpActiveRoom(null);
        addLog("合戦部屋が見つからないか、解散されました。", "system");
      }
    }, (err) => {
      handleFirestoreError(err, OperationType.GET, `artifacts/${APP_ID}/pvpRooms/${roomId}`);
    });
  };

  const ensureDeckFilled = (deckIdx: number): (string | null)[] => {
    const currentDeck = decks[deckIdx];
    const hasAnyGeneral = currentDeck.some(id => id !== null);
    if (!hasAnyGeneral && ownedGenerals.length > 0) {
      const sortedGenerals = [...ownedGenerals]
        .sort((a, b) => (b.atk + b.def) - (a.atk + a.def))
        .slice(0, 5);
      
      const newDeck = [null, null, null, null, null] as (string | null)[];
      sortedGenerals.forEach((g, i) => {
        newDeck[i] = g.uniqueId;
      });
      
      const nextDecks = [...decks];
      nextDecks[deckIdx] = newDeck;
      setDecks(nextDecks);
      addLog("部隊に武将が編成されていなかったため、所持武将から自動的に編成しました！", "success");
      return newDeck;
    }
    return currentDeck;
  };

  const handleCreatePvpRoom = async () => {
    if (!user) {
      addLog("オンライン対戦の作成にはログインが必要です。", "error");
      return;
    }
    
    const activeDeck = ensureDeckFilled(pvpSelectedDeckIdx);
    const targetDeckObjects = activeDeck.map((id) =>
      id ? ownedGenerals.find((g) => g.uniqueId === id) || null : null,
    );
    const drumMultiplier = activeBuff === "drum" ? 1.5 : 1.0;
    const deckPower = targetDeckObjects.reduce((sum, g) => {
      if (!g) return sum;
      const levelMultiplier = 1 + (g.level - 1) * 0.1;
      const breakthroughMultiplier = 1 + (g.breakthrough || 0) * 0.1;
      const allianceBuffMultiplier = 1 + (allianceInfo.level - 1) * 0.05;
      let baseStat = (g.atk + g.int + g.def) * breakthroughMultiplier;
      const weaponId = equippedWeapons[g.uniqueId];
      if (weaponId) {
        const wp = MASTER_WEAPONS.find((w) => w.id === weaponId);
        if (wp) baseStat += wp.atk + wp.int + wp.def;
      }
      return sum + Math.floor(baseStat * levelMultiplier * allianceBuffMultiplier * drumMultiplier);
    }, 0);

    if (deckPower <= 10) {
      addLog("部隊に武将が編制されていません。編制してから再度お試しください。", "error");
      return;
    }
    setPvpMatchmaking(true);
    try {
      const roomId = generate7DigitCode();
      const finalRoomName = pvpRoomNameInput.trim() || `${pvpCustomName || user.displayName || "殿"}の特設陣屋`;
      
      const generalsInSelectedDeck = targetDeckObjects
        .filter((g): g is any => g !== null)
        .map(g => ({
          name: g.name,
          rarity: g.rarity,
          level: g.level,
          faction: g.faction,
          atk: g.atk,
          int: g.int,
          def: g.def,
        }));

      const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", roomId);
      const initialRoomData = {
        roomId,
        roomName: finalRoomName,
        status: "waiting",
        hostUid: user.uid,
        hostName: pvpCustomName || user.displayName || "名無しの名将",
        hostPower: deckPower,
        hostGenerals: generalsInSelectedDeck,
        hostTactic: "",
        hostHp: deckPower,
        guestUid: null,
        guestName: null,
        guestPower: null,
        guestGenerals: null,
        guestTactic: "",
        guestHp: 0,
        turn: 1,
        winnerUid: null,
        roundLogs: [`合戦部屋【${finalRoomName}】が新設されました。部屋コード: ${roomId}。いざ尋常に勝負！`],
        updatedAt: serverTimestamp(),
      };

      await setDoc(docRef, initialRoomData);
      setPvpRoomNameInput("");
      addLog(`合戦部屋「${finalRoomName}」を開きました。部屋コード【${roomId}】を他プレイヤーに配りましょう！`, "success");
      
      subscribeToRoom(roomId);
    } catch (err: any) {
      handleFirestoreError(err, OperationType.WRITE, `artifacts/${APP_ID}/pvpRooms`);
      addLog(`合戦部屋の作成に失敗しました: ${err.message}`, "error");
    } finally {
      setPvpMatchmaking(false);
    }
  };

  const handleJoinByCode = async () => {
    if (!user) {
      addLog("参戦にはログインが必要です。", "error");
      return;
    }
    const code = pvpJoinCodeInput.trim();
    if (!code || code.length !== 7 || isNaN(Number(code))) {
      addLog("合戦部屋コードは7桁の半角数字で入力してください。", "error");
      return;
    }
    
    const activeDeck = ensureDeckFilled(pvpSelectedDeckIdx);
    const targetDeckObjects = activeDeck.map((id) =>
      id ? ownedGenerals.find((g) => g.uniqueId === id) || null : null,
    );
    const drumMultiplier = activeBuff === "drum" ? 1.5 : 1.0;
    const deckPower = targetDeckObjects.reduce((sum, g) => {
      if (!g) return sum;
      const levelMultiplier = 1 + (g.level - 1) * 0.1;
      const breakthroughMultiplier = 1 + (g.breakthrough || 0) * 0.1;
      const allianceBuffMultiplier = 1 + (allianceInfo.level - 1) * 0.05;
      let baseStat = (g.atk + g.int + g.def) * breakthroughMultiplier;
      const weaponId = equippedWeapons[g.uniqueId];
      if (weaponId) {
        const wp = MASTER_WEAPONS.find((w) => w.id === weaponId);
        if (wp) baseStat += wp.atk + wp.int + wp.def;
      }
      return sum + Math.floor(baseStat * levelMultiplier * allianceBuffMultiplier * drumMultiplier);
    }, 0);

    if (deckPower <= 10) {
      addLog("部隊に武将が編制されていません。編制してから再度お試しください。", "error");
      return;
    }

    setPvpMatchmaking(true);
    try {
      const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", code);
      const snapshot = await getDoc(docRef);
      if (!snapshot.exists()) {
        addLog(`指定された部屋（コード: ${code}）が見つかりません。番号をお確かめください。`, "error");
        setPvpMatchmaking(false);
        return;
      }
      
      const room: any = { ...snapshot.data(), id: snapshot.id };
      if (room.status !== "waiting") {
        addLog("その合戦部屋はすでに満員か、合戦が開始されています。", "error");
        setPvpMatchmaking(false);
        return;
      }

      if (room.hostUid === user.uid) {
        addLog("己の部屋には参戦できません。相手の参戦を待ちましょう。", "error");
        setPvpMatchmaking(false);
        return;
      }

      await handleJoinPvpRoom(room);
      setPvpJoinCodeInput("");
    } catch (err: any) {
      addLog(`部屋コードでの参戦に失敗しました: ${err.message}`, "error");
    } finally {
      setPvpMatchmaking(false);
    }
  };

  const handleJoinPvpRoom = async (room: any) => {
    if (!user) {
      addLog("参戦にはログインが必要です。", "error");
      return;
    }
    
    const activeDeck = ensureDeckFilled(pvpSelectedDeckIdx);
    const targetDeckObjects = activeDeck.map((id) =>
      id ? ownedGenerals.find((g) => g.uniqueId === id) || null : null,
    );
    const drumMultiplier = activeBuff === "drum" ? 1.5 : 1.0;
    const deckPower = targetDeckObjects.reduce((sum, g) => {
      if (!g) return sum;
      const levelMultiplier = 1 + (g.level - 1) * 0.1;
      const breakthroughMultiplier = 1 + (g.breakthrough || 0) * 0.1;
      const allianceBuffMultiplier = 1 + (allianceInfo.level - 1) * 0.05;
      let baseStat = (g.atk + g.int + g.def) * breakthroughMultiplier;
      const weaponId = equippedWeapons[g.uniqueId];
      if (weaponId) {
        const wp = MASTER_WEAPONS.find((w) => w.id === weaponId);
        if (wp) baseStat += wp.atk + wp.int + wp.def;
      }
      return sum + Math.floor(baseStat * levelMultiplier * allianceBuffMultiplier * drumMultiplier);
    }, 0);

    if (deckPower <= 10) {
      addLog("部隊に武将が編制されていません。編制してから再度お試しください。", "error");
      return;
    }
    if (room.hostUid === user.uid) {
      addLog("己の部屋には参戦できません。相手の参戦を待ちましょう。", "error");
      return;
    }
    setPvpMatchmaking(true);
    try {
      const generalsInSelectedDeck = targetDeckObjects
        .filter((g): g is any => g !== null)
        .map(g => ({
          name: g.name,
          rarity: g.rarity,
          level: g.level,
          faction: g.faction,
          atk: g.atk,
          int: g.int,
          def: g.def,
        }));

      const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", room.roomId);
      
      await setDoc(docRef, {
        ...room,
        guestUid: user.uid,
        guestName: pvpCustomName || user.displayName || "名無しの参謀",
        guestPower: deckPower,
        guestGenerals: generalsInSelectedDeck,
        guestHp: deckPower,
        guestTactic: "",
        status: "ready",
        roundLogs: [...room.roundLogs, `【参戦】${pvpCustomName || user.displayName || "名無しの参謀"}が部隊（戦力: ${deckPower.toLocaleString()}）を率いて合戦に参戦しました！`],
        updatedAt: serverTimestamp(),
      });
      
      addLog(`「${room.roomName}」に参戦しました！いざ、対戦開始！`, "success");
      subscribeToRoom(room.roomId);
    } catch (err: any) {
      handleFirestoreError(err, OperationType.WRITE, `artifacts/${APP_ID}/pvpRooms/${room.roomId}`);
      addLog(`参戦に失敗しました: ${err.message}`, "error");
    } finally {
      setPvpMatchmaking(false);
    }
  };

  const handleClosePvpRoom = async (roomId: string) => {
    try {
      const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", roomId);
      await deleteDoc(docRef);
      if (pvpUnsubscribeRef.current) {
        pvpUnsubscribeRef.current();
        pvpUnsubscribeRef.current = null;
      }
      setPvpActiveRoom(null);
      addLog("合戦部屋を解散しました。", "system");
    } catch (err: any) {
      handleFirestoreError(err, OperationType.DELETE, `artifacts/${APP_ID}/pvpRooms/${roomId}`);
      addLog("解散に失敗しました。", "error");
    }
  };

  const handleLeavePvpRoom = async (room: any) => {
    if (pvpUnsubscribeRef.current) {
      pvpUnsubscribeRef.current();
      pvpUnsubscribeRef.current = null;
    }
    setPvpActiveRoom(null);

    try {
      const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", room.roomId);
      await setDoc(docRef, {
        ...room,
        guestUid: null,
        guestName: null,
        guestPower: null,
        guestGenerals: null,
        guestHp: 0,
        guestTactic: "",
        status: "waiting",
        roundLogs: [...room.roundLogs, `対戦相手が陣営から退却しました。合戦は待機状態に戻ります。`],
        updatedAt: serverTimestamp(),
      });
      addLog("対戦から退却しました。", "system");
    } catch (err: any) {
      handleFirestoreError(err, OperationType.WRITE, `artifacts/${APP_ID}/pvpRooms/${room.roomId}`);
    }
  };

  const handleStartCpuBattle = async () => {
    if (!user) {
      addLog("模擬戦の開始にはログインが必要です。", "error");
      return;
    }
    
    const activeDeck = ensureDeckFilled(pvpSelectedDeckIdx);
    const targetDeckObjects = activeDeck.map((id) =>
      id ? ownedGenerals.find((g) => g.uniqueId === id) || null : null,
    );
    const drumMultiplier = activeBuff === "drum" ? 1.5 : 1.0;
    const deckPower = targetDeckObjects.reduce((sum, g) => {
      if (!g) return sum;
      const levelMultiplier = 1 + (g.level - 1) * 0.1;
      const breakthroughMultiplier = 1 + (g.breakthrough || 0) * 0.1;
      const allianceBuffMultiplier = 1 + (allianceInfo.level - 1) * 0.05;
      let baseStat = (g.atk + g.int + g.def) * breakthroughMultiplier;
      const weaponId = equippedWeapons[g.uniqueId];
      if (weaponId) {
        const wp = MASTER_WEAPONS.find((w) => w.id === weaponId);
        if (wp) baseStat += wp.atk + wp.int + wp.def;
      }
      return sum + Math.floor(baseStat * levelMultiplier * allianceBuffMultiplier * drumMultiplier);
    }, 0);

    if (deckPower <= 10) {
      addLog("部隊に武将が編制されていません。編制してから再度お試しください。", "error");
      return;
    }
    setPvpMatchmaking(true);
    try {
      const roomId = generate7DigitCode();
      const finalRoomName = `${pvpCustomName || user.displayName || "殿"} vs 仮想敵将`;
      
      const cpuPower = Math.max(100, Math.floor(deckPower * (0.85 + Math.random() * 0.3)));
      const cpuGenerals = [
        { name: "仮：木曽義仲", rarity: "SSR", level: 50, faction: "源氏", atk: 180, int: 110, def: 150 },
        { name: "仮：巴御前", rarity: "SR", level: 45, faction: "源氏", atk: 160, int: 90, def: 130 },
        { name: "仮：樋口兼光", rarity: "HR", level: 40, faction: "源氏", atk: 140, int: 100, def: 120 }
      ];

      const generalsInSelectedDeck = targetDeckObjects
        .filter((g): g is any => g !== null)
        .map(g => ({
          name: g.name,
          rarity: g.rarity,
          level: g.level,
          faction: g.faction,
          atk: g.atk,
          int: g.int,
          def: g.def,
        }));

      const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", roomId);
      const initialRoomData = {
        roomId,
        roomName: finalRoomName,
        status: "ready",
        hostUid: user.uid,
        hostName: pvpCustomName || user.displayName || "名無しの名将",
        hostPower: deckPower,
        hostGenerals: generalsInSelectedDeck,
        hostTactic: "",
        hostHp: deckPower,
        guestUid: "cpu",
        guestName: "仮想敵将・木曽義仲",
        guestPower: cpuPower,
        guestGenerals: cpuGenerals,
        guestTactic: "",
        guestHp: cpuPower,
        turn: 1,
        winnerUid: null,
        roundLogs: [`模擬合戦【${finalRoomName}】が開始されました。いざ尋常に勝負！`],
        updatedAt: serverTimestamp(),
      };

      await setDoc(docRef, initialRoomData);
      setPvpRoomNameInput("");
      addLog(`仮想敵将との模擬戦を開始しました！`, "success");
      
      subscribeToRoom(roomId);
    } catch (err: any) {
      handleFirestoreError(err, OperationType.WRITE, `artifacts/${APP_ID}/pvpRooms`);
      addLog(`模擬戦の作成に失敗しました: ${err.message}`, "error");
    } finally {
      setPvpMatchmaking(false);
    }
  };

  const handleInjectCpuGuest = async (room: any) => {
    if (!user) return;
    setPvpMatchmaking(true);
    try {
      const cpuPower = Math.max(100, Math.floor(room.hostPower * (0.85 + Math.random() * 0.3)));
      const cpuGenerals = [
        { name: "仮：木曽義仲", rarity: "SSR", level: 50, faction: "源氏", atk: 180, int: 110, def: 150 },
        { name: "仮：巴御前", rarity: "SR", level: 45, faction: "源氏", atk: 160, int: 90, def: 130 },
        { name: "仮：樋口兼光", rarity: "HR", level: 40, faction: "源氏", atk: 140, int: 100, def: 120 }
      ];

      const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", room.roomId);
      await setDoc(docRef, {
        ...room,
        guestUid: "cpu",
        guestName: "仮想敵将・木曽義仲",
        guestPower: cpuPower,
        guestGenerals: cpuGenerals,
        guestHp: cpuPower,
        guestTactic: "",
        status: "ready",
        roundLogs: [...room.roundLogs, `【参戦】仮想敵将・木曽義仲（戦力: ${cpuPower.toLocaleString()}）が急襲、模擬合戦が開始されました！`],
        updatedAt: serverTimestamp(),
      });
      addLog("仮想敵将を召喚し、模擬戦を開始しました！", "success");
    } catch (err: any) {
      addLog("CPU召喚に失敗しました。", "error");
    } finally {
      setPvpMatchmaking(false);
    }
  };

  const handleSelectPvpTactic = async (tactic: string) => {
    if (!pvpActiveRoom || !user) return;
    const isHost = pvpActiveRoom.hostUid === user.uid;
    const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", pvpActiveRoom.roomId);

    let updatedData = { ...pvpActiveRoom, updatedAt: serverTimestamp() };
    if (isHost) {
      updatedData.hostTactic = tactic;
      if (pvpActiveRoom.guestUid === "cpu") {
        const tactics = ["charge", "ambush", "defend", "fire"];
        updatedData.guestTactic = tactics[Math.floor(Math.random() * tactics.length)];
      }
    } else {
      updatedData.guestTactic = tactic;
    }

    try {
      await setDoc(docRef, updatedData);
      soundManager.playClick();
    } catch (err: any) {
      handleFirestoreError(err, OperationType.WRITE, `artifacts/${APP_ID}/pvpRooms/${pvpActiveRoom.roomId}`);
    }
  };

  const handleCancelReadyState = async () => {
    if (!pvpActiveRoom || !user) return;
    const isHost = pvpActiveRoom.hostUid === user.uid;
    const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", pvpActiveRoom.roomId);

    let updatedData = { ...pvpActiveRoom, updatedAt: serverTimestamp() };
    if (isHost) {
      updatedData.hostTactic = "";
    } else {
      updatedData.guestTactic = "";
    }

    try {
      await setDoc(docRef, updatedData);
      soundManager.playClick();
    } catch (err: any) {
      handleFirestoreError(err, OperationType.WRITE, `artifacts/${APP_ID}/pvpRooms/${pvpActiveRoom.roomId}`);
    }
  };

  const getGeneralSkill = (g: any, side: string, turn: number) => {
    if (!g) return null;
    
    // Deterministic but feels random based on stats and turn
    const seed = (g.atk || 100) + (g.int || 100) + (g.def || 100) + turn;
    const isTriggered = (seed % 3) === 0; // 33% trigger rate for high-drama skills
    if (!isTriggered) return null;

    const name = g.name || "名無しの豪傑";
    const isHighInt = (g.int || 100) >= 130;
    const isHighAtk = (g.atk || 100) >= 150;
    const isHighDef = (g.def || 100) >= 135;

    if (name.includes("巴御前")) {
      return {
        name: "巴御前の強弓 🏹",
        effectText: "神速の強弓より放たれた大矢が敵の本陣を鋭く貫き、戦局を圧倒する！",
        dmgBonusMult: 1.4,
        dmgReductionMult: 1.0
      };
    }
    if (name.includes("木曽義仲")) {
      return {
        name: "火牛の計 🔥🐂",
        effectText: "角に松明を灯した数百頭の牛群が地響きを立てて敵陣を蹂躙、壊滅的な大打撃を与える！",
        dmgBonusMult: 1.45,
        dmgReductionMult: 0.9
      };
    }
    if (name.includes("静御前")) {
      return {
        name: "白拍子の舞 🌸✨",
        effectText: "戦場に咲く一輪の華のごとき優美な舞！味方兵卒の士気を極限まで鼓舞し、受ける痛打を軽減する！",
        dmgBonusMult: 1.2,
        dmgReductionMult: 0.75
      };
    }
    if (name.includes("源義経")) {
      return {
        name: "八艘飛び 🦅🌊",
        effectText: "波濤を渡る変幻自在の跳躍！敵の守備網を完全に無力化し、防護を無視して大打撃を加える！",
        dmgBonusMult: 1.5,
        dmgReductionMult: 1.0
      };
    }
    if (name.includes("弁慶")) {
      return {
        name: "五条大橋の仁王立ち 🛡️👹",
        effectText: "巨躯を盾に一歩も引かず！敵軍の苛烈な猛攻を一身に受け止め、全軍の被ダメージを半減する！",
        dmgBonusMult: 0.9,
        dmgReductionMult: 0.5
      };
    }
    if (name.includes("那須与一")) {
      return {
        name: "扇の的射抜き 🎯🏹",
        effectText: "「南無八幡大菩薩…！」祈りと共に放たれた矢は、一矢違わず敵陣の急所を撃ち抜いた！",
        dmgBonusMult: 1.4,
        dmgReductionMult: 1.0
      };
    }

    // Fallbacks based on high-end attributes
    if (isHighInt) {
      return {
        name: "神算鬼謀の奇策 💡📜",
        effectText: "敵の軍略・気流の動きをも完全に看破。知略を尽くした包囲陣を敷き、主導権を奪う！",
        dmgBonusMult: 1.3,
        dmgReductionMult: 0.85
      };
    }
    if (isHighAtk) {
      return {
        name: "一騎当千の猛勇 ⚔️🦁",
        effectText: "雄叫び一つ、修羅の如き突破力で敵前衛をなぎ払い、鮮烈なる痛撃を浴びせる！",
        dmgBonusMult: 1.35,
        dmgReductionMult: 1.0
      };
    }
    if (isHighDef) {
      return {
        name: "金城鉄壁の布陣 🏰🧱",
        effectText: "一糸乱れぬ楯列を組み上げ、敵の鋭い攻勢を跳ね返す盤石の防衛線を構築する！",
        dmgBonusMult: 1.0,
        dmgReductionMult: 0.65
      };
    }

    // Default triggers by rarity
    if (g.rarity === "SSR") {
      return {
        name: "武家の極み 🌟",
        effectText: "名門の圧倒的な威光。全軍に檄を飛ばし、覇気みなぎる突進を開始する！",
        dmgBonusMult: 1.25,
        dmgReductionMult: 0.9
      };
    } else if (g.rarity === "SR") {
      return {
        name: "名将の采配 🚩",
        effectText: "堅実な合戦指揮。隙のない布陣転換により、攻守の均衡を最適化する！",
        dmgBonusMult: 1.15,
        dmgReductionMult: 0.95
      };
    }

    return null;
  };

  const resolvePvpRound = async (room: any) => {
    if (room.hostUid !== user?.uid || room.status !== "ready") return;
    
    const hT = room.hostTactic;
    const gT = room.guestTactic;
    if (!hT || !gT) return;

    const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", room.roomId);

    const tacticNames: Record<string, string> = {
      charge: "突撃 ⚔️ (Charge)",
      ambush: "奇襲 🦅 (Ambush)",
      defend: "堅守 🛡️ (Block)",
      fire: "火計 🔥 (Fire)"
    };

    let hostMult = 1.0;
    let guestMult = 1.0;
    let clashLog = `【第${room.turn}合戦の激突！】`;

    if (hT === gT) {
      clashLog += ` 互いに【${tacticNames[hT]}】を展開！互角の真っ向勝負となり、両軍とも一歩も譲らず戦線は膠着状態に！`;
    } else if (
      (hT === "charge" && gT === "ambush") ||
      (hT === "ambush" && gT === "fire") ||
      (hT === "fire" && gT === "defend") ||
      (hT === "defend" && gT === "charge")
    ) {
      hostMult = 1.6;
      guestMult = 0.4;
      clashLog += ` ［東軍 ${room.hostName}］が軍配【${tacticNames[hT]}】にて、［西軍 ${room.guestName}］の【${tacticNames[gT]}】の弱点を鮮やかに急襲！戦局を完全支配！`;
    } else {
      hostMult = 0.4;
      guestMult = 1.6;
      clashLog += ` ［西軍 ${room.guestName}］が軍配【${tacticNames[gT]}】にて、［東軍 ${room.hostName}］の【${tacticNames[hT]}】の意図を完全に看破し、包囲陣で封殺！`;
    }

    // Get active commander general for each side for this turn
    const hostGen = room.hostGenerals && room.hostGenerals.length > 0
      ? room.hostGenerals[(room.turn - 1) % room.hostGenerals.length]
      : null;
    const guestGen = room.guestGenerals && room.guestGenerals.length > 0
      ? room.guestGenerals[(room.turn - 1) % room.guestGenerals.length]
      : null;

    const hostSkill = getGeneralSkill(hostGen, "host", room.turn);
    const guestSkill = getGeneralSkill(guestGen, "guest", room.turn);

    let hostSkillAtkMult = 1.0;
    let hostSkillDefMult = 1.0;
    let guestSkillAtkMult = 1.0;
    let guestSkillDefMult = 1.0;

    if (hostSkill && hostGen) {
      hostSkillAtkMult = hostSkill.dmgBonusMult;
      hostSkillDefMult = hostSkill.dmgReductionMult;
      clashLog += `\n🌟 ［東軍］先陣・${hostGen.name}が奥義【${hostSkill.name}】を発動！『${hostSkill.effectText}』`;
    }
    if (guestSkill && guestGen) {
      guestSkillAtkMult = guestSkill.dmgBonusMult;
      guestSkillDefMult = guestSkill.dmgReductionMult;
      clashLog += `\n🌟 ［西軍］先陣・${guestGen.name}が奥義【${guestSkill.name}】を発動！『${guestSkill.effectText}』`;
    }

    // Critical strike chance (10%)
    const hostCrit = Math.random() < 0.15;
    const guestCrit = Math.random() < 0.15;
    const hostCritMult = hostCrit ? 1.5 : 1.0;
    const guestCritMult = guestCrit ? 1.5 : 1.0;

    // Calculate final damage values
    const baseHostDmg = Math.floor(
      (room.hostPower * 0.15 + 450) * 
      hostMult * 
      hostSkillAtkMult * 
      guestSkillDefMult * 
      hostCritMult *
      (0.85 + Math.random() * 0.3)
    );

    const baseGuestDmg = Math.floor(
      (room.guestPower * 0.15 + 450) * 
      guestMult * 
      guestSkillAtkMult * 
      hostSkillDefMult * 
      guestCritMult *
      (0.85 + Math.random() * 0.3)
    );

    const newGuestHp = Math.max(0, room.guestHp - baseHostDmg);
    const newHostHp = Math.max(0, room.hostHp - baseGuestDmg);

    if (hostCrit) {
      clashLog += `\n💥 【会心の一撃！】［東軍］の猛烈な一撃が敵の喉元を捉えた！`;
    }
    if (guestCrit) {
      clashLog += `\n💥 【会心の一撃！】［西軍］の電撃的カウンターが牙をむく！`;
    }

    clashLog += `\n⚔️ ［東軍 ${room.hostName}］は「${baseHostDmg.toLocaleString()}」のダメージをたたき込み、［西軍 ${room.guestName}］は「${baseGuestDmg.toLocaleString()}」の痛烈な反撃を浴びせました！`;

    let nextStatus = "ready";
    let winnerUid = null;
    if (newHostHp === 0 && newGuestHp === 0) {
      nextStatus = "finished";
      clashLog += `\n🎌 【合戦決着】両陣営、一歩も退かぬ凄絶な攻防の末、共に力尽き相打ち（引き分け）となりました！`;
    } else if (newGuestHp === 0) {
      nextStatus = "finished";
      winnerUid = room.hostUid;
      clashLog += `\n🎌 【合戦決着】総大将［${room.hostName}］の天下無双の軍略が冴え渡り、敵軍を総崩れに追い込んで大勝利を収めました！`;
    } else if (newHostHp === 0) {
      nextStatus = "finished";
      winnerUid = room.guestUid;
      clashLog += `\n🎌 【合戦決着】総大将［${room.guestName}］の電撃布陣が功を奏し、敵陣営の本陣を見事に撃破、圧倒的勝利を飾りました！`;
    } else {
      nextStatus = "clashed";
    }

    try {
      await setDoc(docRef, {
        ...room,
        hostHp: newHostHp,
        guestHp: newGuestHp,
        status: nextStatus,
        winnerUid,
        roundLogs: [...room.roundLogs, clashLog],
        updatedAt: serverTimestamp(),
      });
    } catch (err: any) {
      handleFirestoreError(err, OperationType.WRITE, `artifacts/${APP_ID}/pvpRooms/${room.roomId}`);
    }
  };

  const loadData = async (uid: string) => {
    try {
      const docRef = doc(
        db,
        "artifacts",
        APP_ID,
        "users",
        uid,
        "saveData",
        "main",
      );
      const docSnap = await getDoc(docRef);
      if (docSnap.exists()) {
        const data = docSnap.data();
        if (data.gold !== undefined) setGold(data.gold);
        if (data.copper !== undefined) setCopper(data.copper);
        if (data.peoplesLoyalty !== undefined)
          setPeoplesLoyalty(data.peoplesLoyalty);
        if (data.inventory) setInventory(data.inventory);
        if (data.inventoryWeapons) setInventoryWeapons(data.inventoryWeapons);
        if (data.equippedWeapons) setEquippedWeapons(data.equippedWeapons);
        if (data.ownedGenerals) setOwnedGenerals(data.ownedGenerals);
        if (data.occupiedLands) setOccupiedLands(data.occupiedLands);
        if (data.allianceInfo) setAllianceInfo(data.allianceInfo);
        if (data.hasNavy !== undefined) setHasNavy(data.hasNavy);
        if (data.navyShips) setNavyShips(data.navyShips);
        if (data.diplomacyRelations)
          setDiplomacyRelations(data.diplomacyRelations);
        if (data.LANDS) {
          setLands(data.LANDS);
          const satsuma = data.LANDS.find((l: any) => l.id === 1);
          if (satsuma) setSelectedLand(satsuma);
        }
        if (data.capitalLandId !== undefined) {
          setCapitalLandId(data.capitalLandId);
        } else {
          setCapitalLandId(1);
        }
        if (data.decks) setDecks(JSON.parse(data.decks));
        if (data.items) setItems(data.items);
        if (data.activeBuff !== undefined) setActiveBuff(data.activeBuff);
        addLog("記帳（セーブデータ）を読み込みました。", "system");
      } else {
        // Initialize default generals
        const initGeneral = {
          ...MASTER_GENERALS.find((g) => g.id === "g10"),
          uniqueId: generateId(),
          level: 1,
          currentExp: 0,
          breakthrough: 0,
        };
        const initPeasant = {
          ...MASTER_GENERALS.find((g) => g.id === "g31"),
          uniqueId: generateId(),
          level: 1,
          currentExp: 0,
          breakthrough: 0,
        };
        setOwnedGenerals([initGeneral, initPeasant]);
        setDecks([
          [initGeneral.uniqueId, initPeasant.uniqueId, null, null, null],
          [null, null, null, null, null],
          [null, null, null, null, null],
        ]);
        setOccupiedLands([]);
      }
    } catch (error) {
      handleFirestoreError(error, OperationType.GET, "saveData");
    }
  };

  const handleSave = async () => {
    if (!user) {
      addLog("保存にはログインが必要です。", "error");
      return;
    }
    setIsSaving(true);
    try {
      const docRef = doc(
        db,
        "artifacts",
        APP_ID,
        "users",
        user.uid,
        "saveData",
        "main",
      );
      await setDoc(docRef, {
        gold,
        copper,
        peoplesLoyalty,
        inventory,
        inventoryWeapons,
        equippedWeapons,
        ownedGenerals,
        decks: JSON.stringify(decks),
        occupiedLands,
        items,
        activeBuff,
        allianceInfo,
        hasNavy,
        navyShips,
        diplomacyRelations,
        LANDS: lands,
        capitalLandId,
        updatedAt: serverTimestamp(),
      });
      addLog("現状を記帳いたしました。（セーブ完了）", "success");
    } catch (error) {
      handleFirestoreError(error, OperationType.WRITE, "saveData");
    } finally {
      setIsSaving(false);
    }
  };

  const handleResetGame = async () => {
    try {
      setIsSaving(true);
      const initGeneral = {
        ...MASTER_GENERALS.find((g) => g.id === "g10"),
        uniqueId: generateId(),
        level: 1,
        currentExp: 0,
        breakthrough: 0,
      };
      const initPeasant = {
        ...MASTER_GENERALS.find((g) => g.id === "g31"),
        uniqueId: generateId(),
        level: 1,
        currentExp: 0,
        breakthrough: 0,
      };

      const defaultDecks = [
        [initGeneral.uniqueId, initPeasant.uniqueId, null, null, null],
        [null, null, null, null, null],
        [null, null, null, null, null],
      ];

      const defaultGold = 20000;
      const defaultCopper = 25000;
      const defaultPeoplesLoyalty = 100;
      const defaultInventory = {
        tactics: 5,
        drum: 3,
        hora: 2,
      };
      const defaultInventoryWeapons = { wp1: 2, wp2: 1, wp3: 1 };
      const defaultEquippedWeapons = {};
      const defaultOwnedGenerals = [initGeneral, initPeasant];
      const defaultOccupiedLands: number[] = [];
      const defaultItems: any[] = [];
      const defaultActiveBuff = null;
      const defaultAllianceInfo = {
        name: "天下布武の会",
        level: 3,
        exp: 1500,
        maxExp: 5000,
        members: [
          { name: "御館様 (あなた)", power: 0, role: "盟主" },
          { name: "柴田権六", power: 10500, role: "副盟主" },
          { name: "木下藤吉郎", power: 8900, role: "構成員" },
        ],
        logs: [
          "木下藤吉郎が同盟に加入しました。",
        ],
      };
      const defaultHasNavy = false;
      const defaultNavyShips = {
        sengokubune: 0,
        kobaya: 0,
        sekibune: 0,
        atakebune: 0,
      };
      const defaultDiplomacyRelations = {};
      const defaultLands = INITIAL_LANDS;
      const defaultCapitalLandId = 1;

      if (user) {
        const docRef = doc(
          db,
          "artifacts",
          APP_ID,
          "users",
          user.uid,
          "saveData",
          "main",
        );
        await setDoc(docRef, {
          gold: defaultGold,
          copper: defaultCopper,
          peoplesLoyalty: defaultPeoplesLoyalty,
          inventory: defaultInventory,
          inventoryWeapons: defaultInventoryWeapons,
          equippedWeapons: defaultEquippedWeapons,
          ownedGenerals: defaultOwnedGenerals,
          decks: JSON.stringify(defaultDecks),
          occupiedLands: defaultOccupiedLands,
          items: defaultItems,
          activeBuff: defaultActiveBuff,
          allianceInfo: defaultAllianceInfo,
          hasNavy: defaultHasNavy,
          navyShips: defaultNavyShips,
          diplomacyRelations: defaultDiplomacyRelations,
          LANDS: defaultLands,
          capitalLandId: defaultCapitalLandId,
          updatedAt: serverTimestamp(),
        });
      }

      setGold(defaultGold);
      setCopper(defaultCopper);
      setPeoplesLoyalty(defaultPeoplesLoyalty);
      setInventory(defaultInventory);
      setInventoryWeapons(defaultInventoryWeapons);
      setEquippedWeapons(defaultEquippedWeapons);
      setOwnedGenerals(defaultOwnedGenerals);
      setDecks(defaultDecks);
      setOccupiedLands(defaultOccupiedLands);
      setItems(defaultItems);
      setActiveBuff(defaultActiveBuff);
      setAllianceInfo(defaultAllianceInfo);
      setHasNavy(defaultHasNavy);
      setNavyShips(defaultNavyShips);
      setDiplomacyRelations(defaultDiplomacyRelations);
      setLands(defaultLands);
      setCapitalLandId(defaultCapitalLandId);

      const satsuma = defaultLands.find((l) => l.id === 1);
      if (satsuma) setSelectedLand(satsuma);

      setBattleScene(null);
      setIsGameOver(false);
      setIsGameClear(false);
      setShowResetModal(false);

      addLog("天下布武の志を新たに、初めから出陣しました！", "success");
    } catch (error) {
      console.error(error);
      addLog("リセットに失敗しました。", "error");
    } finally {
      setIsSaving(false);
    }
  };

  // --- Game Logic ---

  const isReachable = (landId: number) => {
    if (occupiedLands.includes(landId)) return true;

    // レベル3以下の城は、進路の接続関係に関わらず常に進軍可能としてストーリーを進められるようにする
    const targetLand = lands.find((l) => l.id === landId);
    if (targetLand && targetLand.level <= 3) return true;

    if (occupiedLands.length === 0) return landId === 1;

    // 自領または同盟領を通過可能とする
    const walkable = new Set<number>(occupiedLands);
    lands.forEach((l) => {
      if (diplomacyRelations[l.owner] === "alliance") {
        walkable.add(l.id);
      }
    });

    // BFSで探索
    const queue: number[] = [...occupiedLands];
    const visited = new Set<number>(occupiedLands);

    while (queue.length > 0) {
      const current = queue.shift()!;
      if (current === landId) return true;

      // 隣接ノードを探す
      const neighbors: number[] = [];
      ROUTE_CONNECTIONS.forEach(([a, b]) => {
        if (a === current) neighbors.push(b);
        if (b === current) neighbors.push(a);
      });

      for (const neighbor of neighbors) {
        if (neighbor === landId) return true; // 隣接していれば到達可能

        // neighborが自領か同盟領(walkable)であり、未訪問なら、そこを通過できる
        if (walkable.has(neighbor) && !visited.has(neighbor)) {
          visited.add(neighbor);
          queue.push(neighbor);
        }
      }
    }

    return false;
  };

  const triggerPeasantRebellion = () => {
    const landsCount = Math.max(occupiedLands.length, 1);
    const lostCopper = Math.min(copper, landsCount * 1500);
    const lostGold = Math.min(gold, landsCount * 150);

    setCopper((prev) => Math.max(prev - lostCopper, 0));
    setGold((prev) => Math.max(prev - lostGold, 0));

    let rebellionMsg = "";
    const lossCandidates = occupiedLands.filter(
      (id) => id !== 1 && id !== capitalLandId,
    );

    if (lossCandidates.length > 0) {
      const randomIndex = Math.floor(Math.random() * lossCandidates.length);
      const lostLandId = lossCandidates[randomIndex];
      const lostLand = lands.find((l) => l.id === lostLandId);
      const lostLandName = lostLand ? lostLand.name.split(" ")[0] : "領地";

      setOccupiedLands((prev) => prev.filter((id) => id !== lostLandId));
      setPeoplesLoyalty((prev) => Math.min(prev + 20, 100));

      rebellionMsg = `暴動を鎮圧するため【${lostLandName}】の支配権を失い、復興費用等として銅銭 -${lostCopper.toLocaleString()}、小判 -${lostGold.toLocaleString()} の甚大な被害が出ました！`;
    } else {
      rebellionMsg = `大規模な暴動が発生！ 一揆を鎮圧・懐柔するため、武力闘争により銅銭 -${lostCopper.toLocaleString()}、小判 -${lostGold.toLocaleString()} が浪費されました。`;
    }

    addLog(
      `【一揆勃発！】苛政に耐えかねた農民たちが各地で一揆（暴動）を起こしました！`,
      "error",
    );
    addLog(`【一揆被害】${rebellionMsg}`, "error");
  };

  const handleCollectTax = (policy: "generous" | "standard" | "harsh") => {
    const landsCount = Math.max(occupiedLands.length, 1);
    const peasantCount = ownedGenerals.filter((g) => g.name === "農民").length;

    let goldReward = 0;
    let copperReward = 0;
    let loyaltyChange = 0;
    let baseRebellionChance = 0;
    let policyName = "";

    if (policy === "generous") {
      policyName = "寛大（三公七民）";
      copperReward = peasantCount * 1000;
      goldReward = landsCount * 80;
      loyaltyChange = 5;
      baseRebellionChance = 0;
    } else if (policy === "standard") {
      policyName = "標準（五公五民）";
      copperReward = peasantCount * 2500;
      goldReward = landsCount * 220;
      loyaltyChange = -15;
      baseRebellionChance = 12;
    } else if (policy === "harsh") {
      policyName = "苛烈（八公二民）";
      copperReward = peasantCount * 6000;
      goldReward = landsCount * 550;
      loyaltyChange = -35;
      baseRebellionChance = 45;
    }

    let extraRebellionChance = 0;
    if (peoplesLoyalty < 25) {
      extraRebellionChance = 45;
    } else if (peoplesLoyalty < 55) {
      extraRebellionChance = 20;
    }

    // Coastal harbors and maritime logistics (Sengokubune) boost
    const activePortCount = lands.filter(
      (l) => l.isPort && occupiedLands.includes(l.id),
    ).length;
    const basePortIncomeCopper = activePortCount * 1200;
    const basePortIncomeGold = activePortCount * 100;

    const sengokubuneCount = navyShips?.sengokubune || 0;
    const tradeBoostMultiplier = 1 + sengokubuneCount * 0.1; // +10% revenue boost per Sengokubune

    const finalCopperReward = Math.floor(
      (copperReward + basePortIncomeCopper) * tradeBoostMultiplier,
    );
    const finalGoldReward = Math.floor(
      (goldReward + basePortIncomeGold) * tradeBoostMultiplier,
    );

    const finalRebellionChance = Math.min(
      baseRebellionChance + extraRebellionChance,
      100,
    );
    const newLoyalty = Math.max(
      Math.min(peoplesLoyalty + loyaltyChange, 100),
      0,
    );

    setPeoplesLoyalty(newLoyalty);
    setGold((prev) => prev + finalGoldReward);
    setCopper((prev) => prev + finalCopperReward);

    let taxSuccessLog = `【徴税】${policyName}の方針で税を徴収しました。銅銭 +${finalCopperReward.toLocaleString()}、小判 +${finalGoldReward.toLocaleString()}。`;
    if (activePortCount > 0 || sengokubuneCount > 0) {
      taxSuccessLog += ` （内、支配海港 ${activePortCount} 港の交易＆海運物流増益を含む）`;
    }
    addLog(taxSuccessLog, "success");

    const roll = Math.random() * 100;
    if (roll < finalRebellionChance) {
      triggerPeasantRebellion();
    } else {
      if (loyaltyChange < 0) {
        addLog(
          `【民政】農民たちの間に不満が広がっています（民忠: ${newLoyalty}）。`,
          "warning",
        );
      } else {
        addLog(
          `【民政】農民たちは慈悲深い徳政を讃えています（民忠: ${newLoyalty}）。`,
          "success",
        );
      }

      // 徴税時の敵襲（カウンター攻撃）判定
      const invasionChance =
        policy === "harsh" ? 30 : policy === "standard" ? 15 : 0;
      if (invasionChance > 0 && Math.random() * 100 < invasionChance) {
        setTimeout(() => {
          triggerInvasionCheck();
        }, 1500);
      }
    }
  };

  const triggerInvasionCheck = (passedOccupied?: number[]) => {
    const list = passedOccupied || occupiedLands;
    // 支配している領地がない場合は判定しない
    if (list.length === 0) return;

    // プレイヤーが支配している領地からランダムに1つ襲撃対象を選ぶ
    const targetId = list[Math.floor(Math.random() * list.length)];
    const targetLand = lands.find((l) => l.id === targetId);
    if (!targetLand) return;

    // 隣接する未占領地を特定し、そこを支配している大名を襲撃者候補にする
    const neighborUnoccupiedLandIds = ROUTE_CONNECTIONS.filter(
      ([a, b]) =>
        (a === targetId && !list.includes(b)) ||
        (b === targetId && !list.includes(a)),
    ).map(([a, b]) => (a === targetId ? b : a));

    let attackerFaction = "群";
    if (neighborUnoccupiedLandIds.length > 0) {
      const neighborLands = lands.filter((l) =>
        neighborUnoccupiedLandIds.includes(l.id),
      );
      const validNeighbors = neighborLands.filter(
        (l) => l.owner && l.owner !== "群",
      );
      if (validNeighbors.length > 0) {
        attackerFaction =
          validNeighbors[Math.floor(Math.random() * validNeighbors.length)]
            .owner;
      } else {
        attackerFaction =
          neighborLands[Math.floor(Math.random() * neighborLands.length)]
            .owner || "群";
      }
    } else {
      const rivalDaimyos = [
        "織田",
        "武田",
        "上杉",
        "毛利",
        "伊達",
        "北条",
        "大友",
        "長宗我部",
        "今川",
      ];
      attackerFaction =
        rivalDaimyos[Math.floor(Math.random() * rivalDaimyos.length)];
    }

    // 同盟大名からの侵攻は発生しない
    if (diplomacyRelations[attackerFaction] === "alliance") {
      return;
    }

    const basePower = targetLand.power || 1000;
    // 対象土地の難度・規模に基づき、襲撃勢力を算出
    const enemyPower = Math.floor(basePower * (0.8 + Math.random() * 0.6));

    setInvasionEvent({
      land: targetLand,
      enemyFaction: attackerFaction,
      enemyPower: Math.max(enemyPower, 400),
    });

    addLog(
      `【急告・敵襲】${attackerFaction}軍（兵力 ${Math.max(enemyPower, 400).toLocaleString()}）が、我らが領土【${targetLand.name.split(" ")[0]}】を狙って侵略軍を興しました！`,
      "error",
    );
  };

  const handleGiveAlms = () => {
    const cost = 2500;
    if (copper < cost) {
      addLog(
        "【民政】施しを行うための銅銭（2,500）が不足しています。",
        "error",
      );
      return;
    }

    setCopper((prev) => prev - cost);
    setPeoplesLoyalty((prev) => Math.min(prev + 25, 100));
    addLog(
      `【民政】貯蔵された米や汁を農民に分け与えました！民忠が 25 上昇しました。`,
      "success",
    );
  };

  const handleUseItem = (item: any) => {
    if (item.effect === "exp_all") {
      setOwnedGenerals((prev) =>
        prev.map((g) => {
          let newExp = g.currentExp + 1000;
          let newLevel = g.level;
          while (newExp >= newLevel * 100) {
            newExp -= newLevel * 100;
            newLevel++;
          }
          return { ...g, level: newLevel, currentExp: newExp };
        }),
      );
      soundManager.playLevelUp();
      addLog(`【兵法】兵法書を使用。全武将の経験値が上昇した！`, "success");
    } else if (item.effect === "power_boost") {
      setActiveBuff("drum");
      soundManager.playLevelUp();
      addLog(
        `【陣太鼓】陣太鼓を鳴らした！次戦、我が軍の兵力は1.5倍となる！`,
        "rare",
      );
    }

    setItems((prev) => {
      const idx = prev.findIndex((i) => i.uniqueId === item.uniqueId);
      if (idx > -1) {
        const newItems = [...prev];
        newItems.splice(idx, 1);
        return newItems;
      }
      return prev;
    });
  };

  const handleUseItemBulkByItemType = (itemType: string, count: number) => {
    if (count <= 0) return;
    const matchingItems = items.filter((itm) => itm.id === itemType);
    if (matchingItems.length < count) {
      addLog(`【不足】所持している該当道具が足りません。`, "error");
      return;
    }

    const item = matchingItems[0];

    if (item.effect === "exp_all") {
      setOwnedGenerals((prev) =>
        prev.map((g) => {
          let newExp = g.currentExp + 1000 * count;
          let newLevel = g.level;
          while (newExp >= newLevel * 100) {
            newExp -= newLevel * 100;
            newLevel++;
          }
          return { ...g, level: newLevel, currentExp: newExp };
        }),
      );
      soundManager.playLevelUp();
      addLog(`【六韜】兵法書をまとめて ${count} 冊使用。全武将の経験値が合計 ${1000 * count} 大幅に上昇しました！`, "success");
    } else if (item.effect === "power_boost") {
      // Multiple drums still just toggle activeBuff, but consumes the quantity
      setActiveBuff("drum");
      soundManager.playLevelUp();
      addLog(`【陣太鼓】武太鼓を ${count} 個まとめて鳴らした！次戦、我が軍の勢い（兵力）は最高潮に達し1.5倍となる！`, "rare");
    }

    // Remove matching items from items state
    setItems((prev) => {
      let removed = 0;
      return prev.filter((itm) => {
        if (itm.id === itemType && removed < count) {
          removed++;
          return false;
        }
        return true;
      });
    });
  };

  const handleBuyItem = (wp: any, quantity: number, currency: "gold" | "copper") => {
    if (quantity <= 0) return;
    const isTool = wp.type === "道具";
    const goldPrice = wp.price * quantity;
    const copperPrice = (wp.price * 5) * quantity;

    if (currency === "gold") {
      if (gold < goldPrice) {
        addLog(`【不足】小判が足りません。(${goldPrice.toLocaleString()}小判必要)`, "error");
        return;
      }
      setGold((prev) => prev - goldPrice);
    } else {
      if (copper < copperPrice) {
        addLog(`【不足】銅銭が足りません。(${copperPrice.toLocaleString()}銅銭必要)`, "error");
        return;
      }
      setCopper((prev) => prev - copperPrice);
    }

    // Add weapons to inventoryWeapons
    if (!isTool) {
      setInventoryWeapons((prev) => ({
        ...prev,
        [wp.id]: (prev[wp.id] || 0) + quantity,
      }));
    }

    // Add items (and/or weapons) to items array for user inventory display
    setItems((prev) => {
      const added: any[] = [];
      for (let i = 0; i < quantity; i++) {
        added.push({ ...wp, uniqueId: generateId() });
      }
      return [...prev, ...added];
    });

    soundManager.playClick();
    addLog(
      `【買付】${wp.name} を ${quantity} 個、${currency === "gold" ? "小判" : "銅銭"}で購入しました！`,
      "success",
    );
  };

  const equipWeapon = (gUniqueId: string, wpId: string) => {
    const currentWpId = equippedWeapons[gUniqueId];

    setInventoryWeapons((prev) => {
      const next = { ...prev };
      // Decrease the new weapon count
      if (next[wpId] && next[wpId] > 0) {
        next[wpId] -= 1;
      }
      // Return old weapon if any
      if (currentWpId) {
        next[currentWpId] = (next[currentWpId] || 0) + 1;
      }
      return next;
    });

    setEquippedWeapons((prev) => ({
      ...prev,
      [gUniqueId]: wpId,
    }));

    soundManager.playLevelUp();
    const wpObj = MASTER_WEAPONS.find((w) => w.id === wpId);
    if (wpObj) {
      addLog(`【兵装】武将に ${wpObj.name} を配備・装備させました。戦闘力が加算されます。`, "success");
    }
  };

  const unequipWeapon = (gUniqueId: string) => {
    const currentWpId = equippedWeapons[gUniqueId];
    if (!currentWpId) return;

    setInventoryWeapons((prev) => ({
      ...prev,
      [currentWpId]: (prev[currentWpId] || 0) + 1,
    }));

    setEquippedWeapons((prev) => {
      const next = { ...prev };
      delete next[gUniqueId];
      return next;
    });

    const wpObj = MASTER_WEAPONS.find((w) => w.id === currentWpId);
    if (wpObj) {
      addLog(`【兵装】${wpObj.name} の装備を解除し、兵装蔵に戻しました。`, "system");
    }
  };

  const handleBulkGiveBooks = (gUniqueId: string, count: number) => {
    const totalBooks = items.filter((itm) => itm.id === "item1").length;
    if (count <= 0 || totalBooks < count) {
      addLog(`【不足】授与するための兵法書が足りません。`, "error");
      return;
    }

    // Consume books from items list
    setItems((prev) => {
      let removed = 0;
      return prev.filter((itm) => {
        if (itm.id === "item1" && removed < count) {
          removed++;
          return false;
        }
        return true;
      });
    });

    // Boost General EXP
    setOwnedGenerals((prev) =>
      prev.map((g) => {
        if (g.uniqueId !== gUniqueId) return g;
        let newExp = g.currentExp + 1000 * count;
        let newLevel = g.level;
        while (newExp >= newLevel * 100) {
          newExp -= newLevel * 100;
          newLevel++;
        }
        return { ...g, level: newLevel, currentExp: newExp };
      }),
    );

    // If trainingGeneral is currently state-associated, keep stats in sync
    setTrainingGeneral((prev: any) => {
      if (!prev || prev.uniqueId !== gUniqueId) return prev;
      let newExp = prev.currentExp + 1000 * count;
      let newLevel = prev.level;
      while (newExp >= newLevel * 100) {
        newExp -= newLevel * 100;
        newLevel++;
      }
      return { ...prev, level: newLevel, currentExp: newExp };
    });

    soundManager.playLevelUp();
    addLog(`【兵法書一括授与】兵法書をまとめて ${count} 冊授与！武将が大きく力量を伸ばしました！`, "success");
  };

  const calculateDeckPower = (deckIdx: number) => {
    const targetDeck = decks[deckIdx].map((id) =>
      id ? ownedGenerals.find((g) => g.uniqueId === id) || null : null,
    );
    const drumMultiplier = activeBuff === "drum" ? 1.5 : 1.0;

    return targetDeck.reduce((sum, g) => {
      if (!g) return sum;
      const levelMultiplier = 1 + (g.level - 1) * 0.1;
      const breakthroughMultiplier = 1 + (g.breakthrough || 0) * 0.1;
      const allianceBuffMultiplier = 1 + (allianceInfo.level - 1) * 0.05;
      let baseStat = (g.atk + g.int + g.def) * breakthroughMultiplier;
      const weaponId = equippedWeapons[g.uniqueId];
      if (weaponId) {
        const wp = MASTER_WEAPONS.find((w) => w.id === weaponId);
        if (wp) baseStat += wp.atk + wp.int + wp.def;
      }
      return (
        sum +
        Math.floor(
          baseStat *
            levelMultiplier *
            allianceBuffMultiplier *
            drumMultiplier *
            10,
        )
      );
    }, 0);
  };

  const drawGacha = (times: number) => {
    const cost = times === 1 ? 200 : 1900;
    if (gold < cost) {
      addLog(`小判が不足しておる。(${cost}小判必要)`, "error");
      return;
    }
    setGold((prev) => prev - cost);
    const newGenerals: any[] = [];
    const drawLogs: any[] = [];

    for (let i = 0; i < times; i++) {
      const rand = Math.random() * 100;
      let rarity = 1;
      if (rand < 5) rarity = 5;
      else if (rand < 20) rarity = 4;
      else if (rand < 50) rarity = 3;
      else if (rand < 80) rarity = 2;

      const candidates = MASTER_GENERALS.filter((g) => g.rarity === rarity);
      const selected =
        candidates[Math.floor(Math.random() * candidates.length)];
      const newGeneral = {
        ...selected,
        uniqueId: generateId(),
        level: 1,
        currentExp: 0,
        breakthrough: 0,
      };
      newGenerals.push(newGeneral);

      let logStr = `登用: [星${rarity}] ${selected.name} が配下に加わった！`;
      if (rarity === 5) logStr = `【大吉報】` + logStr;
      drawLogs.push({
        id: generateId(),
        text: logStr,
        type: rarity === 5 ? "rare" : "normal",
      });
    }

    setOwnedGenerals((prev) => [...prev, ...newGenerals]);
    setLogs((prev) => [...prev, ...drawLogs]);
    soundManager.playGacha();
    setGachaResult(newGenerals);
  };

  const getEquippedDeckIndex = (uniqueId: string) =>
    decks.findIndex((deck) => deck.includes(uniqueId));
  const isGeneralEquipped = (uniqueId: string) =>
    getEquippedDeckIndex(uniqueId) !== -1;

  const startBattle = (
    land: any,
    deckIdx: number,
    isDefense = false,
    customEnemyPower?: number,
    customEnemyName?: string,
  ) => {
    if (battlingLandId) return;

    if (diplomacyRelations[land.owner] === "alliance") {
      addLog(`【同盟】${land.owner}とは同盟を結んでおる。`, "error");
      return;
    }

    const activeDeckGenerals = decks[deckIdx]
      .map((id) =>
        id ? ownedGenerals.find((g) => g.uniqueId === id) || null : null,
      )
      .filter((g) => g !== null);

    if (activeDeckGenerals.length === 0) {
      addLog(`部隊に誰も出陣しておらんぞ！`, "error");
      return;
    }

    setBattlingLandId(land.id);
    if (isDefense) setInvasionEvent(null);

    if (
      !isDefense &&
      land.owner &&
      land.owner !== "群" &&
      diplomacyRelations[land.owner] !== "alliance"
    ) {
      setDiplomacyRelations((prev) => ({ ...prev, [land.owner]: "hostile" }));
    }

    const deckPower = calculateDeckPower(deckIdx);

    // 隣接する同盟領地からの援軍を計算
    const neighborAllianceLands = lands.filter((l) => {
      const isNeighbor = ROUTE_CONNECTIONS.some(([a, b]) => {
        return (a === land.id && b === l.id) || (b === land.id && a === l.id);
      });
      return isNeighbor && diplomacyRelations[l.owner] === "alliance";
    });

    let allianceReinforcements = 0;
    const reinforcementLogs: string[] = [];
    if (neighborAllianceLands.length > 0) {
      neighborAllianceLands.forEach((al) => {
        const powerAdded = Math.floor(al.power * 0.25 + deckPower * 0.2);
        allianceReinforcements += powerAdded;
        reinforcementLogs.push(
          `【同盟】隣接する${al.name}（${al.owner}家）より兵力 ${powerAdded.toLocaleString()} の援軍が合流！`,
        );
      });
    }

    // Coastal navy support firing (焙烙・大筒)
    let navyPowerAdded = 0;
    if (hasNavy) {
      const kobayaCount = navyShips?.kobaya || 0;
      const sekibuneCount = navyShips?.sekibune || 0;
      const atakebuneCount = navyShips?.atakebune || 0;
      navyPowerAdded =
        1000 + kobayaCount * 100 + sekibuneCount * 350 + atakebuneCount * 1200;
      reinforcementLogs.push(
        `【海軍】我が国の直属水軍（関船・安宅船など）が焙烙玉と大筒で猛烈な沿岸援護射撃を実行！ 兵力 +${navyPowerAdded.toLocaleString()} 相当の有利！`,
      );
    }

    // 居城防衛の補正
    let capitalDefensePowerAdded = 0;
    if (isDefense && land.id === capitalLandId) {
      capitalDefensePowerAdded = Math.floor(deckPower * 0.4 + 2000);
      reinforcementLogs.push(
        `【居城防衛補正】我が国の本城・居城【${land.name.split(" ")[0]}】に籠城！城兵と領民が総立ちとなり、防衛戦闘力 +${capitalDefensePowerAdded.toLocaleString()} の必死の加勢！`,
      );
    }

    const totalDeckPower = deckPower + allianceReinforcements + navyPowerAdded + capitalDefensePowerAdded;
    const targetEnemyPower = customEnemyPower || land.power;
    const targetEnemyName =
      customEnemyName ||
      (occupiedLands.includes(land.id) ? "賊軍" : `${land.owner}軍`);

    setBattleScene({
      phase: "intro",
      land,
      activeDeck: activeDeckGenerals,
      deckPower: totalDeckPower, // 援軍を加算した総戦闘力
      baseDeckPower: deckPower,
      allianceReinforcements,
      reinforcementLogs,
      enemyPower: targetEnemyPower,
      enemyName: targetEnemyName,
      resultText: "",
      isVictory: false,
      isDraw: false,
      isDefense,
      selectedTactic: null,
      clashStep: 0,
      clashLogs: [],
    });
  };

  const launchTacticalSiege = async (tacticKey: string) => {
    if (!battleScene) return;

    const {
      deckPower: originalDeckPower,
      enemyPower: originalEnemyPower,
      land,
      isDefense,
      activeDeck: activeDeckGenerals,
      enemyName: targetEnemyName,
      enemyPower: targetEnemyPower
    } = battleScene;

    let finalDeckPower = originalDeckPower;
    let finalEnemyPower = originalEnemyPower;
    let tacticMsg = "";

    // Apply tactic specific changes
    if (tacticKey === "surrender_immediate") {
      finalDeckPower = Math.floor(finalDeckPower * 3.0);
      finalEnemyPower = 1;
      tacticMsg = "【一括制圧・無血開城】天下比類なき我が軍威に敵守備兵は戦わずして全面降伏！静かに城門が開かれました。";
    } else if (tacticKey === "gambit_suicide") {
      finalDeckPower = Math.floor(finalDeckPower * 2.1);
      tacticMsg = "【背水の陣・乾坤一擲】退路を塞ぎ、命を擲って戦う決死の背水の陣！全軍が限界を大きく超越した破壊力を発揮！(自軍総戦闘力 +110% / 逆転勝鬨特典)";
    } else if (tacticKey === "assault") {
      finalDeckPower = Math.floor(finalDeckPower * 1.15);
      tacticMsg = "【魚鱗の陣・突撃】全兵士が一丸となり正面大突破の陣。全員 of 鬨の声と共に、大手城壁へ突貫！(武将戦闘力 +15%)";
    } else if (tacticKey === "crane") {
      finalEnemyPower = Math.floor(finalEnemyPower * 0.90);
      tacticMsg = "【鶴翼の陣・包囲】大きく左右の翼を広げ城郭を完全包囲。敵の迎撃ルートを遮り、防衛連携を崩す！(敵守備力 -10%)";
    } else if (tacticKey === "horoku") {
      finalDeckPower = Math.floor(finalDeckPower * 1.35);
      if (inventory.drum > 0) {
        setInventory(prev => ({ ...prev, drum: Math.max(0, (prev.drum || 0) - 1) }));
        tacticMsg = "【焙烙大筒面射撃】[陣太鼓]を激しく連打消費！沿岸の安宅船大筒隊が一斉掃射を開始し、城壁の武者窓を粉砕！(自軍戦闘力 +35%)";
      } else if (inventory.tactics > 0) {
        setInventory(prev => ({ ...prev, tactics: Math.max(0, (prev.tactics || 0) - 1) }));
        tacticMsg = "【焙烙大筒面射撃】[兵法書]の奥義を布令消費！焙烙火矢を天守閣へ集中投射、城内を大炎上に追い込む！(自軍戦闘力 +35%)";
      } else {
        setCopper(prev => Math.max(0, prev - 3000));
        tacticMsg = "【焙烙大筒面射撃】国庫より軍費3,000両を調達、傭兵大筒隊を急襲配備し炸裂弾を城壁に撃ち込む！(自軍戦闘力 +35%)";
      }
    } else if (tacticKey === "decoy") {
      finalDeckPower = Math.floor(finalDeckPower * 1.25);
      if (inventory.hora > 0) {
        setInventory(prev => ({ ...prev, hora: Math.max(0, (prev.hora || 0) - 1) }));
        tacticMsg = "【陽動作戦・虚実乱撃】[法螺貝]を他方で猛烈に吹鳴消費！搦手口から忍び衆が静かに侵入。城内は大混乱に陥る！(自軍戦闘力 +25% / 武具獲得チャンス向上)";
      } else {
        setCopper(prev => Math.max(0, prev - 2000));
        tacticMsg = "【陽動作戦・虚実乱撃】即席の別働隊に国費2,000両を持たせ別曲輪で大火を起こさせ、防衛主力を陽動！(自軍戦闘力 +25%)";
      }
    } else {
      // 兵糧攻め / starvation (costs 1200 copper)
      finalEnemyPower = Math.floor(finalEnemyPower * 0.82);
      setCopper(prev => Math.max(0, prev - 1200));
      tacticMsg = "【兵糧乱攻め】国費1,200両を用いて水道と城への補給路を遮断。飢餓と疲弊により、城兵の防闘能力が著しく低下！(敵守備力 -18%)";
    }

    setBattleScene((prev: any) => ({
      ...prev,
      phase: "clash",
      deckPower: finalDeckPower,
      enemyPower: finalEnemyPower,
      selectedTactic: tacticKey,
      clashStep: 0,
      clashLogs: [tacticMsg],
    }));
  };

  const advanceClashDecision = async (decisionKey: string) => {
    if (!battleScene) return;

    let narrative = "";
    let deckMod = 0;
    let enemyMod = 0;
    const currentStep = battleScene.clashStep || 0;

    if (currentStep === 0) {
      if (decisionKey === "front_gate") {
        narrative = "【大手大門強襲】我が軍、大手正面へ大木の破城槌（衝車）を引き出し、渾身の力で突撃！『ズドォーン！』と大地を揺るがす地響きと共に巨大城門を粉砕！ (自軍戦闘力 +10%)";
        deckMod = Math.floor(battleScene.deckPower * 0.10);
      } else if (decisionKey === "back_gate") {
        narrative = "【搦手裏門急襲】守備が手薄な城の裏口「搦手門（裏門）」へ忍び衆と同盟案内の手引きで影から急襲！『ガチャリ』と閂を外して門を大開きにして猛爆進撃！ (敵守備力 -10%)";
        enemyMod = -Math.floor(battleScene.enemyPower * 0.10);
      } else {
        narrative = "【外濠水路潜入】外濠の隠し排水門より、甲冑を脱ぎ捨てて潜行進軍！城壁の裏口へ攀攀潜入し、奇襲の火起しに成功！ (敵守備力 -8%)";
        enemyMod = -Math.floor(battleScene.enemyPower * 0.08);
      }
    } else if (currentStep === 1) {
      if (decisionKey === "first_ward") {
        narrative = "【一の丸高地突破】「我が軍続け、一の丸の一の門を奪うのだ！」鉄砲の雨と一の曲輪の急斜面を押し返し、一の丸の防御盾を叩き伏せて突破に成功！ (自軍戦闘力 +10%)";
        deckMod = Math.floor(battleScene.deckPower * 0.10);
      } else if (decisionKey === "dry_moat") {
        narrative = "【空堀急斜面攀登】一の丸を側面から完璧に迂回！傾斜角度60度を超える恐怖の空堀（からぼり）の断崖を綱・梯子を架けて急攀、上空の側守陣を奇襲破砕！ (敵守備力 -10%)";
        enemyMod = -Math.floor(battleScene.enemyPower * 0.10);
      } else {
        narrative = "【櫓窓焙烙連射】一の丸を狙い撃つ高楼矢倉に対し、火矢と炸裂焙烙玉を窓穴に放り込む！大炎上が生じて敵兵は消火と瓦解で大パニックに！ (自軍戦闘力 +8%)";
        deckMod = Math.floor(battleScene.deckPower * 0.08);
      }
    } else if (currentStep === 2) {
      if (decisionKey === "second_ward_assault") {
        narrative = "【二の丸千鳥強襲】二の丸防衛陣を前に、一斉の鉄砲斉射を張りつつ全力抜刀！超巨大な『千鳥門』の二重扉を粉砕し、本丸の廊下橋までを一気に制圧！ (自軍戦闘力 +12%)";
        deckMod = Math.floor(battleScene.deckPower * 0.12);
      } else if (decisionKey === "ninja_infiltrate") {
        narrative = "【二の丸抜け穴手引き】城内の『隠し井戸』『地下抜け道』を配下の草（忍び）が手引き。二の丸の裏裏勝手口に忍び込んで一斉放火を放ち、二の丸防御を無力化！ (敵守備力 -12%)";
        enemyMod = -Math.floor(battleScene.enemyPower * 0.12);
      } else {
        narrative = "【虚実大騒動・時の声】「時の声を上げよ！」全山に法螺貝と陣太鼓の乱打音を鳴り響かせ、東口より猛爆突破と見せかけて、防衛が手薄になった西門から強襲！ (敵守備力 -8%)";
        enemyMod = -Math.floor(battleScene.enemyPower * 0.08);
      }
    } else if (currentStep === 3) {
      if (decisionKey === "duel") {
        narrative = "【本丸一騎打ち要求】大天守閣の本陣扉を蹴り倒し、大将が堂々の一騎討ちを要求！「我が武勇の刃、受けてみよ！」凄まじい鉄火と真剣白刃の末、敵将を討ち伏す！ (自軍戦闘力 +15%)";
        deckMod = Math.floor(battleScene.deckPower * 0.15);
      } else if (decisionKey === "flame_arrows") {
        narrative = "【天守焦熱爆破】「全大筒・火矢、天守閣の最上階へ放て！」鯱鉾を焦がす炎の嵐を巻き起こし、本丸天守閣を急爆炎上させ敵将を敗北に追いやる！ (敵守備力 -15%)";
        enemyMod = -Math.floor(battleScene.enemyPower * 0.15);
      } else {
        narrative = "【本丸総包囲降伏勧告】「勝負は決した、兵員の命を無駄にするな！」天守閣を完璧に幾重もの矢ぶすまで完全包囲！死を悟った敵軍が最後は城門を開け無血開城！ (敵守備力 -12%)";
        enemyMod = -Math.floor(battleScene.enemyPower * 0.12);
      }
    }

    const nextStep = currentStep + 1;
    const nextDeckPower = Math.max(10, battleScene.deckPower + deckMod);
    const nextEnemyPower = Math.max(10, battleScene.enemyPower + enemyMod);

    setBattleScene((prev: any) => {
      if (!prev) return prev;
      return {
        ...prev,
        clashStep: nextStep,
        deckPower: nextDeckPower,
        enemyPower: nextEnemyPower,
        clashLogs: [...(prev.clashLogs || []), narrative],
      };
    });

    if (nextStep >= 4) {
      setTimeout(() => {
        setBattleScene((prev: any) => {
          if (!prev) return prev;
          resolveFinalBattleOutcome(nextDeckPower, nextEnemyPower, prev);
          return prev;
        });
      }, 1000);
    }
  };

  const resolveFinalBattleOutcome = async (
    finalDeckPower: number,
    finalEnemyPower: number,
    currentBattleScene: any
  ) => {
    if (!currentBattleScene) return;

    const {
      land,
      isDefense,
      activeDeck: activeDeckGenerals,
      enemyName: targetEnemyName,
      enemyPower: targetEnemyPower,
      selectedTactic: tacticKey
    } = currentBattleScene;

    const tacticMsg = tacticKey === "surrender_immediate"
      ? "一括制圧・無血開城勧告"
      : tacticKey === "gambit_suicide"
        ? "決死の背水の陣総爆進"
        : tacticKey === "assault"
          ? "魚鱗の正面総突撃"
          : tacticKey === "crane"
            ? "鶴翼の包囲網攻勢"
            : tacticKey === "horoku"
              ? "焙烙大筒の猛烈射撃"
              : tacticKey === "decoy"
                ? "陽動作戦・搦手虚実攻"
                : "水道遮断兵糧攻め";

    let finalPower = finalDeckPower * (0.8 + Math.random() * 0.4);
    
    // Guaranteed victory for bloodless surrender
    if (tacticKey === "surrender_immediate") {
      finalPower = finalEnemyPower * 10;
    }

    let isVictory = finalPower > finalEnemyPower;
    let isDraw = !isVictory && finalPower > finalEnemyPower * 0.7;
    let resultText = isVictory
      ? isDefense
        ? "防衛成功"
        : "大勝・占領"
      : isDraw
        ? "引き分け"
        : isDefense
          ? "防衛失敗"
          : "大敗";

    if (isVictory) {
      soundManager.playVictory();
    } else if (!isDraw) {
      soundManager.playDefeat();
    }

    let lootList: any[] = [];
    const suicideMultiplier = tacticKey === "gambit_suicide" ? 2 : 1;

    if (isVictory) {
      if (isDefense) {
        const defCopper = Math.floor(targetEnemyPower / 10) * suicideMultiplier;
        setCopper((prev) => prev + defCopper);
        lootList.push({ type: "copper", name: "銅銭", count: defCopper });
      } else {
        const isAlreadyOccupied = occupiedLands.includes(land.id);
        if (!isAlreadyOccupied) {
          const goldRew = (land.goldReward || 0) * suicideMultiplier;
          const copperRew = (land.reward || 0) * suicideMultiplier;
          setGold((prev) => prev + goldRew);
          setCopper((prev) => prev + copperRew);
          if (copperRew > 0) lootList.push({ type: "copper", name: "銅銭", count: copperRew });
          if (goldRew > 0) lootList.push({ type: "gold", name: "小判", count: goldRew });
          setOccupiedLands((prev) => {
            const next = [...prev, land.id];
            if (next.length === lands.length) setIsGameClear(true);
            return next;
          });
        }
      }

      // 1. Drop Items
      const itemsDroppedNum = Math.floor(Math.random() * 2) + 1; // 1~2
      const tempDropItems: Record<string, number> = {};
      for (let i = 0; i < itemsDroppedNum; i++) {
        const rand = Math.random();
        let itemKey = "tactics";
        let itemName = "兵法書";
        if (rand < 0.45) {
          itemKey = "tactics";
          itemName = "兵法書";
        } else if (rand < 0.80) {
          itemKey = "drum";
          itemName = "陣太鼓";
        } else {
          itemKey = "hora";
          itemName = "法螺貝";
        }
        tempDropItems[itemKey] = (tempDropItems[itemKey] || 0) + 1;
        lootList.push({ type: "item", id: itemKey, name: itemName, count: 1 });
      }

      setInventory((prev) => {
        const next = { ...prev };
        Object.entries(tempDropItems).forEach(([key, val]) => {
          next[key] = (next[key] || 0) + val;
        });
        return next;
      });

      // 2. Drop Weapon (improved by decoy tactic / guaranteed by suicide gambit)
      const landLevel = land.level || 1;
      const bonusRate = tacticKey === "decoy" ? 0.20 : 0.0;
      const isSuicide = tacticKey === "gambit_suicide";
      const weaponDropChance = isSuicide ? 1.0 : 0.4 + 0.1 * landLevel + bonusRate;
      if (Math.random() < weaponDropChance || landLevel >= 5) {
        const randRarity = isSuicide ? Math.random() * 0.3 : Math.random();
        let weaponId = "wp1";
        let weaponName = "打刀";
        let rarityNum = 3;

        if (landLevel >= 8) {
          if (randRarity < 0.3) {
            const wp5Options = [
              { id: "wp4", name: "名物・童子切安綱", rarity: 5 },
              { id: "wp5", name: "軍配団扇", rarity: 5 },
              { id: "wp7", name: "雑賀八汐筒", rarity: 5 },
              { id: "wp8", name: "南蛮大筒", rarity: 5 },
            ];
            const opt = wp5Options[Math.floor(Math.random() * wp5Options.length)];
            weaponId = opt.id;
            weaponName = opt.name;
            rarityNum = opt.rarity;
          } else if (randRarity < 0.8) {
            const wp4Options = [
              { id: "wp2", name: "十文字槍", rarity: 4 },
              { id: "wp3", name: "種子島", rarity: 4 },
              { id: "wp6", name: "国友筒", rarity: 4 },
            ];
            const opt = wp4Options[Math.floor(Math.random() * wp4Options.length)];
            weaponId = opt.id;
            weaponName = opt.name;
            rarityNum = opt.rarity;
          } else {
            weaponId = "wp1";
            weaponName = "打刀";
            rarityNum = 3;
          }
        } else if (landLevel >= 5) {
          if (randRarity < 0.12) {
            const wp5Options = [
              { id: "wp4", name: "名物・童子切安綱", rarity: 5 },
              { id: "wp5", name: "軍配団扇", rarity: 5 },
              { id: "wp7", name: "雑賀八汐筒", rarity: 5 },
              { id: "wp8", name: "南蛮大筒", rarity: 5 },
            ];
            const opt = wp5Options[Math.floor(Math.random() * wp5Options.length)];
            weaponId = opt.id;
            weaponName = opt.name;
            rarityNum = opt.rarity;
          } else if (randRarity < 0.52) {
            const wp4Options = [
              { id: "wp2", name: "十文字槍", rarity: 4 },
              { id: "wp3", name: "種子島", rarity: 4 },
              { id: "wp6", name: "国友筒", rarity: 4 },
            ];
            const opt = wp4Options[Math.floor(Math.random() * wp4Options.length)];
            weaponId = opt.id;
            weaponName = opt.name;
            rarityNum = opt.rarity;
          } else {
            weaponId = "wp1";
            weaponName = "打刀";
            rarityNum = 3;
          }
        } else {
          if (randRarity < 0.02) {
            weaponId = "wp4";
            weaponName = "名物・童子切安綱";
            rarityNum = 5;
          } else if (randRarity < 0.22) {
            const wp4Options = [
              { id: "wp2", name: "十文字槍", rarity: 4 },
              { id: "wp3", name: "種子島", rarity: 4 },
            ];
            const opt = wp4Options[Math.floor(Math.random() * wp4Options.length)];
            weaponId = opt.id;
            weaponName = opt.name;
            rarityNum = opt.rarity;
          } else {
            weaponId = "wp1";
            weaponName = "打刀";
            rarityNum = 3;
          }
        }

        setInventoryWeapons((prev) => ({
          ...prev,
          [weaponId]: (prev[weaponId] || 0) + 1,
        }));
        lootList.push({ type: "weapon", id: weaponId, name: weaponName, count: 1, rarity: rarityNum });
      }

      // General Exp calculation
      setOwnedGenerals((prev) =>
        prev.map((g) => {
          const isActive = activeDeckGenerals.some(
            (ag: any) => ag.uniqueId === g.uniqueId,
          );
          if (!isActive) return g;
          let newExp = g.currentExp + (isDefense ? land.exp * 2 : land.exp);
          let newLevel = g.level;
          while (newExp >= newLevel * 100) {
            newExp -= newLevel * 100;
            newLevel++;
          }
          return { ...g, level: newLevel, currentExp: newExp };
        }),
      );

      const spoilsText = lootList
        .filter((item) => item.type === "item" || item.type === "weapon")
        .map((item) => `[${item.name}]`)
        .join("、");
      if (spoilsText) {
        addLog(`【戦利品獲得】占領地の財宝庫・武具蔵を捜索し、戦利品 ${spoilsText} を引き揚げました！`, "success");
      }
      if (tacticKey === "gambit_suicide") {
        addLog(`【起死回生の大勝利】敵軍が圧倒的優勢である絶望的な状況のなか「背水の陣」を見事に発動して、歴史に伝わる大逆転大勝を収めました！！`, "success");
      }
    } else if (!isDraw && isDefense) {
      setOccupiedLands((prev) => prev.filter((id) => id !== land.id));
      if (land.id === capitalLandId) setIsGameOver(true);
    }

    setBattleScene((prev: any) => ({
      ...prev,
      phase: "result",
      resultText,
      isVictory,
      isDraw,
      loot: lootList,
    }));
    if (activeBuff === "drum") setActiveBuff(null);

    const summary = await callAI(
      "report",
      `戦国時代の合戦結果です。【指揮作戦】${tacticMsg} 【出陣武将】${activeDeckGenerals.map((g: any) => g.name).join(", ")} 【敵軍】兵力 ${targetEnemyPower} の ${targetEnemyName} 【結果】${resultText}`,
    );
    addLog(`【合戦報告】${summary}`, isVictory ? "success" : "error");
  };

  // --- Scroll & Map Handlers ---

  useEffect(() => {
    logsEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [logs]);

  useEffect(() => {
    chatEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [chatMessages, activeTab]);

  const handleMapMouseDown = (e: React.MouseEvent) => {
    if (!mapContainerRef.current) return;
    setIsDragging(true);
    setDragStart({
      x: e.pageX,
      y: e.pageY,
      scrollLeft: mapContainerRef.current.scrollLeft,
      scrollTop: mapContainerRef.current.scrollTop,
    });
    setDragDistance(0);
  };

  const handleMapMouseMove = (e: React.MouseEvent) => {
    if (!isDragging || !mapContainerRef.current) return;
    e.preventDefault();
    const dx = e.pageX - dragStart.x;
    const dy = e.pageY - dragStart.y;
    mapContainerRef.current.scrollLeft = dragStart.scrollLeft - dx;
    mapContainerRef.current.scrollTop = dragStart.scrollTop - dy;
    setDragDistance((prev) => prev + Math.abs(dx) + Math.abs(dy));
  };

  // --- UI Render ---

  return (
    <div className="flex flex-col h-screen bg-dark-bg text-[#D4D4D8] relative overflow-hidden font-sans">
      {/* Modals */}
      <AnimatePresence>
        {isGameOver && (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            className="absolute inset-0 z-[100] bg-black/95 flex flex-col items-center justify-center backdrop-blur-md"
          >
            <Skull className="w-32 h-32 text-red-700 mb-4 animate-pulse" />
            <h1 className="text-6xl md:text-9xl font-black text-red-600 mb-6 tracking-widest drop-shadow-2xl">
              御家滅亡
            </h1>
            <button
              onClick={() => window.location.reload()}
              className="px-10 py-4 bg-red-950 hover:bg-red-900 text-red-200 border-2 border-red-700 rounded-full font-bold text-xl transition-all"
            >
              やり直す
            </button>
          </motion.div>
        )}

        {trainingGeneral && (() => {
          const g = ownedGenerals.find((og) => og.uniqueId === trainingGeneral.uniqueId) || trainingGeneral;
          const equippedWpId = equippedWeapons[g.uniqueId];
          const equippedWpObj = equippedWpId ? MASTER_WEAPONS.find((w) => w.id === equippedWpId) : null;
          const booksCount = items.filter((itm) => itm.id === "item1").length;

          const availableWeapons = Object.entries(inventoryWeapons)
            .filter(([id, count]) => (count as number) > 0)
            .map(([id, count]) => {
              const info = MASTER_WEAPONS.find((w) => w.id === id);
              return info ? { ...info, count: count as number } : null;
            })
            .filter((wp): wp is any => wp !== null);

          return (
            <motion.div
              initial={{ opacity: 0 }}
              animate={{ opacity: 1 }}
              exit={{ opacity: 0 }}
              className="absolute inset-0 z-[100] bg-black/90 flex items-center justify-center backdrop-blur-md p-4 overflow-y-auto"
            >
              <motion.div
                initial={{ scale: 0.95, y: 15 }}
                animate={{ scale: 1, y: 0 }}
                exit={{ scale: 0.95, y: 15 }}
                className="bg-card-bg border border-amber-500/20 max-w-2xl w-full p-8 rounded shadow-2xl relative font-sans text-stone-200"
              >
                <button
                  onClick={() => setTrainingGeneral(null)}
                  className="absolute top-4 right-4 text-stone-500 hover:text-gold transition-colors p-2"
                >
                  <X className="w-6 h-6" />
                </button>

                <div className="border-b border-white/5 pb-6 mb-6">
                  <div className="flex items-center gap-2 mb-1">
                    <span className="bg-white/5 border border-white/10 px-2 py-0.5 rounded text-[10px] text-stone-400 font-bold uppercase tracking-wider">
                      {g.faction}家
                    </span>
                    <RarityStars count={g.rarity} />
                  </div>
                  <h2 className="text-3xl font-serif font-bold text-white tracking-widest flex items-center gap-3">
                    {g.name} <span className="text-gold font-sans text-lg font-light">養成・兵装房</span>
                  </h2>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
                  <div className="space-y-6">
                    <div className="bg-black/40 border border-white/5 p-4 rounded-sm">
                      <h3 className="text-xs font-bold text-gold tracking-widest uppercase mb-3 font-serif">武将の現状（ステータス）</h3>
                      <div className="grid grid-cols-3 gap-2 text-center text-xs mb-4">
                        <div className="bg-white/5 p-2 rounded">
                          <span className="text-stone-500 block text-[9px]">武力</span>
                          <span className="text-red-400 font-bold font-mono text-sm leading-none mt-1 block">
                            {g.atk + (equippedWpObj?.atk || 0)}
                            {equippedWpObj?.atk ? ` (+${equippedWpObj.atk})` : ""}
                          </span>
                        </div>
                        <div className="bg-white/5 p-2 rounded">
                          <span className="text-stone-500 block text-[9px]">知略</span>
                          <span className="text-blue-400 font-bold font-mono text-sm leading-none mt-1 block">
                            {g.int + (equippedWpObj?.int || 0)}
                            {equippedWpObj?.int ? ` (+${equippedWpObj.int})` : ""}
                          </span>
                        </div>
                        <div className="bg-white/5 p-2 rounded">
                          <span className="text-stone-500 block text-[9px]">統率</span>
                          <span className="text-green-400 font-bold font-mono text-sm leading-none mt-1 block">
                            {g.def + (equippedWpObj?.def || 0)}
                            {equippedWpObj?.def ? ` (+${equippedWpObj.def})` : ""}
                          </span>
                        </div>
                      </div>

                      <div className="text-[11px] space-y-1">
                        <div className="flex justify-between">
                          <span>等級（レベル）:</span>
                          <span className="font-bold text-white">LV.{g.level}</span>
                        </div>
                        <div className="flex justify-between">
                          <span>累計経験値:</span>
                          <span className="font-mono text-stone-400">{g.currentExp} / {g.level * 100}</span>
                        </div>
                        <div className="w-full bg-stone-950 h-1.5 rounded-full overflow-hidden mt-1.5 border border-white/5">
                          <div
                            className="bg-gold h-full transition-all duration-300"
                            style={{ width: `${Math.min(100, (g.currentExp / (g.level * 100)) * 100)}%` }}
                          />
                        </div>
                      </div>
                    </div>

                    <div className="bg-black/40 border border-white/5 p-4 rounded-sm">
                      <h3 className="text-xs font-bold text-gold tracking-widest uppercase mb-2 font-serif flex items-center justify-between">
                        <span>兵法書を授与（一括武将養成）</span>
                        <span className="text-[10px] text-stone-500 font-normal">所持本: {booksCount}冊</span>
                      </h3>
                      <p className="text-[9px] text-stone-400 leading-normal mb-4">
                        兵法書を与えることで、武将の経験値が1冊あたり 1,000 上昇します。一括授与でまとめてレベルを上げられます。
                      </p>

                      {booksCount > 0 ? (
                        <div className="space-y-2">
                          <div className="grid grid-cols-2 gap-2 text-[10px]">
                            <button
                              onClick={() => handleBulkGiveBooks(g.uniqueId, 1)}
                              className="bg-white/5 hover:bg-gold hover:text-black py-2 rounded font-bold border border-white/10 transition-all text-center"
                            >
                              1冊授与（+1,000 EXP）
                            </button>
                            {booksCount >= 5 && (
                              <button
                                onClick={() => handleBulkGiveBooks(g.uniqueId, Math.min(5, booksCount))}
                                className="bg-white/5 hover:bg-gold hover:text-black py-2 rounded font-bold border border-white/10 transition-all text-center"
                              >
                                {Math.min(5, booksCount)}冊授与
                              </button>
                            )}
                            {booksCount >= 10 && (
                              <button
                                onClick={() => handleBulkGiveBooks(g.uniqueId, Math.min(10, booksCount))}
                                className="bg-white/5 hover:bg-gold hover:text-black py-2 rounded font-bold border border-white/10 transition-all text-center"
                              >
                                {Math.min(10, booksCount)}冊授与
                              </button>
                            )}
                            <button
                              onClick={() => handleBulkGiveBooks(g.uniqueId, booksCount)}
                              className="bg-amber-950/20 hover:bg-gold hover:text-black hover:border-gold border border-amber-500/30 text-amber-300 py-2 rounded font-bold transition-all text-center col-span-2 flex items-center justify-center gap-1.5"
                            >
                              🚀 全て一括授与（{booksCount}冊 / +{(booksCount * 1000).toLocaleString()} EXP）
                            </button>
                          </div>
                        </div>
                      ) : (
                        <div className="text-center py-4 bg-white/5 rounded text-[10px] text-stone-600 font-serif">
                          手持ちに兵法書がありません。万屋にて購入可能です。
                        </div>
                      )}
                    </div>
                  </div>

                  <div className="space-y-6 flex flex-col justify-between">
                    <div>
                      <div className="bg-black/40 border border-white/5 p-4 rounded-sm mb-4">
                        <h3 className="text-xs font-bold text-gold tracking-widest uppercase mb-3 font-serif">現在の兵装（装備スロット）</h3>
                        {equippedWpObj ? (
                          <div className="flex items-center justify-between bg-white/5 p-3 rounded border border-white/10">
                            <div>
                              <h4 className="font-bold text-white text-sm">{equippedWpObj.name}</h4>
                              <p className="text-[10px] text-stone-400 mt-1 uppercase">分類: {equippedWpObj.type}</p>
                            </div>
                            <button
                              onClick={() => unequipWeapon(g.uniqueId)}
                              className="bg-red-950/20 hover:bg-red-600 hover:text-black text-red-400 border border-red-500/20 text-[10px] font-bold px-3 py-1.5 rounded transition-all"
                            >
                              装備解除
                            </button>
                          </div>
                        ) : (
                          <div className="text-center py-6 bg-white/5 rounded text-[11px] text-stone-500 font-sans italic border border-dashed border-white/10">
                            兵装は装備されていません
                          </div>
                        )}
                      </div>

                      <div className="bg-black/40 border border-white/5 p-4 rounded-sm">
                        <h3 className="text-xs font-bold text-gold tracking-widest uppercase mb-3 font-serif">予備の兵装（配備可能な装備一覧）</h3>
                        {availableWeapons.length > 0 ? (
                          <div className="space-y-2 max-h-[180px] overflow-y-auto custom-scrollbar pr-1 text-stone-300">
                            {availableWeapons.map((wp: any) => (
                              <div
                                key={wp.id}
                                className="flex items-center justify-between bg-white/5 p-2 rounded text-xs border border-white/5 hover:border-white/10 transition-colors"
                              >
                                <div>
                                  <div className="flex items-center gap-1.5">
                                    <span className="font-bold text-white">{wp.name}</span>
                                    <span className="text-[10px] text-gold font-mono font-bold">x{wp.count}</span>
                                  </div>
                                  <div className="flex gap-2 text-[9px] text-stone-400 mt-0.5">
                                    {wp.atk !== 0 && <span className="text-red-400">武+{wp.atk}</span>}
                                    {wp.int !== 0 && <span className="text-blue-400">知+{wp.int}</span>}
                                    {wp.def !== 0 && <span className="text-green-400 font-bold">統+{wp.def}</span>}
                                  </div>
                                </div>
                                <button
                                  onClick={() => equipWeapon(g.uniqueId, wp.id)}
                                  className="bg-emerald-950/20 hover:bg-emerald-500 hover:text-black text-emerald-400 border border-emerald-500/25 px-2.5 py-1 rounded text-[10px] font-bold transition-all"
                                >
                                  装備
                                </button>
                              </div>
                            ))}
                          </div>
                        ) : (
                          <div className="text-center py-6 text-[10px] text-stone-600 font-serif">
                            配備できる遊休兵装がありません。
                          </div>
                        )}
                      </div>
                    </div>

                    <button
                      onClick={() => setTrainingGeneral(null)}
                      className="w-full bg-stone-900 border border-white/10 py-3 text-[11px] font-bold tracking-widest hover:bg-white hover:text-black transition-all uppercase rounded-sm mt-4"
                    >
                      閉じる
                    </button>
                  </div>
                </div>
              </motion.div>
            </motion.div>
          );
        })()}

        {showResetModal && (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            className="absolute inset-0 z-[100] bg-black/85 flex flex-col items-center justify-center backdrop-blur-sm p-4"
          >
            <motion.div
              initial={{ scale: 0.9, y: 20 }}
              animate={{ scale: 1, y: 0 }}
              exit={{ scale: 0.9, y: 20 }}
              className="bg-[#141414] border border-red-550/30 p-8 rounded-sm max-w-md w-full shadow-2xl relative"
            >
              <div className="absolute top-3 right-3">
                <button
                  onClick={() => setShowResetModal(false)}
                  className="text-stone-500 hover:text-stone-300 transition-colors p-1"
                >
                  <X className="w-5 h-5" />
                </button>
              </div>

              <div className="flex flex-col items-center text-center font-sans">
                <AlertTriangle className="w-16 h-16 text-red-500 mb-4 animate-bounce" />
                <h2 className="text-2xl font-serif font-bold text-red-500 mb-2 tracking-widest">
                  天下統一の旅路をリセット
                </h2>
                <div className="text-xs text-stone-400 font-sans leading-relaxed my-4 text-justify bg-black/40 p-4 rounded border border-white/5 space-y-2">
                  <p>
                    これより、現在のすべての領地、獲得した武将、蓄積した銅銭および小判、内政投資、水軍艦隊をすべて初期状態に戻し、再度『最初から』天下統一を目指します。
                  </p>
                  <p className="text-red-400 font-bold">
                    ※この操作は取り消すことができません。よろしいですか？
                  </p>
                </div>

                <div className="flex gap-4 w-full mt-2 font-serif">
                  <button
                    onClick={() => setShowResetModal(false)}
                    className="flex-1 py-3 bg-stone-900 hover:bg-stone-800 text-stone-300 hover:text-white rounded border border-stone-800 transition-all text-xs font-bold tracking-widest cursor-pointer"
                  >
                    断念
                  </button>
                  <button
                    onClick={() => {
                      handleResetGame();
                    }}
                    className="flex-1 py-3 bg-gradient-to-r from-red-950 to-red-900 hover:from-red-900 hover:to-red-800 text-red-200 hover:text-white rounded border border-red-700 hover:border-red-500 transition-all text-xs font-bold tracking-widest shadow-[0_4px_12px_rgba(239,68,68,0.25)] cursor-pointer"
                  >
                    再起
                  </button>
                </div>
              </div>
            </motion.div>
          </motion.div>
        )}
      </AnimatePresence>

      <header className="bg-section-bg border-b border-white/5 p-4 flex justify-between items-center shadow-2xl z-10">
        <h1 className="text-2xl font-serif font-bold text-gold tracking-[0.2em] uppercase">
          戦国真戦{" "}
          <span className="text-xs text-stone-500 font-sans tracking-normal opacity-50 ml-2">
            ~ Fubu no Michi ~
          </span>
        </h1>

        <div className="flex items-center gap-6">
          {/* Procedural BGM music/sfx controller toggle */}
          <button
            onClick={() => {
              const nextMute = !isMuted;
              setIsMuted(nextMute);
              soundManager.setMute(nextMute);
              if (!nextMute) {
                soundManager.playClick();
              }
            }}
            className="bg-stone-950/45 hover:bg-stone-850 border border-white/5 hover:border-gold/30 px-3 py-2 rounded-sm text-stone-300 hover:text-gold flex items-center gap-2 transition-all cursor-pointer"
            title={isMuted ? "音量を有効化" : "消音"}
          >
            {isMuted ? (
              <VolumeX className="w-3.5 h-3.5 text-stone-500" />
            ) : (
              <Volume2 className="w-3.5 h-3.5 text-gold animate-bounce" />
            )}
            <span className="text-[10px] font-bold tracking-widest uppercase">
              音階: {isMuted ? "消音" : "奏楽中"}
            </span>
          </button>

          <button
            onClick={() => setShowResetModal(true)}
            className="bg-red-950/20 hover:bg-red-950/40 border border-red-900/30 hover:border-red-500/50 px-3 py-2 rounded-sm text-red-300 hover:text-red-100 flex items-center gap-1.5 transition-all cursor-pointer"
            title="天下統一の旅路を最初からやり直します"
          >
            <RefreshCw className="w-3.5 h-3.5" />
            <span className="text-[10px] font-bold tracking-widest uppercase">
              最初から
            </span>
          </button>

          {!user ? (
            <button
              onClick={handleGoogleSignIn}
              className="bg-gold text-black px-4 py-2 rounded-sm text-[10px] font-bold tracking-widest uppercase hover:bg-[#D4A159] transition-all"
            >
              門を叩く（ログイン）
            </button>
          ) : (
            <div className="flex items-center gap-4">
              <span className="text-[10px] text-stone-500 uppercase tracking-widest hidden md:block">
                在位: {user.displayName || "殿"}
              </span>
              <button
                onClick={handleSave}
                disabled={isSaving}
                className="bg-white/5 border border-white/10 px-4 py-2 rounded-sm text-stone-300 hover:text-gold hover:border-gold/50 flex items-center gap-2 transition-all cursor-pointer"
              >
                {isSaving ? (
                  <Activity className="w-3.5 h-3.5 animate-spin" />
                ) : (
                  <Save className="w-3.5 h-3.5 transition-transform group-hover:scale-110" />
                )}
                <span className="text-[10px] font-bold tracking-widest uppercase">
                  記帳
                </span>
              </button>
            </div>
          )}
          <div className="flex items-center gap-3 bg-white/5 px-4 py-2 rounded-sm border border-white/5">
            <Coins className="w-4 h-4 text-orange-500" />
            <span className="font-mono text-sm text-orange-400 font-medium tracking-tight">
              {(copper / 1000).toFixed(1)}k
            </span>
          </div>
          <div className="flex items-center gap-3 bg-white/5 px-4 py-2 rounded-sm border border-white/10 shadow-inner">
            <Crown className="w-4 h-4 text-amber-500" />
            <span className="font-mono text-sm text-gold font-bold tracking-tight">
              {gold.toLocaleString()}
            </span>
          </div>
        </div>
      </header>

      <div className="flex flex-1 overflow-hidden">
        <main className="flex-1 flex flex-col p-6 overflow-y-auto custom-scrollbar relative">
          <nav className="flex gap-1 mb-8 border-b border-white/5 pb-0 overflow-x-auto custom-scrollbar shrink-0">
            {[
              {
                id: "map",
                label: "出陣",
                Icon: MapIcon,
                color: "text-red-500",
                glow: "bg-red-500",
                border: "border-red-500",
              },
              {
                id: "castles",
                label: "城一覧",
                Icon: Castle,
                color: "text-stone-400",
                glow: "bg-stone-400",
                border: "border-stone-500",
              },
              {
                id: "deck",
                label: "編制",
                Icon: Tent,
                color: "text-blue-500",
                glow: "bg-blue-500",
                border: "border-blue-500",
              },
              {
                id: "gacha",
                label: "登用",
                Icon: Users,
                color: "text-amber-500",
                glow: "bg-amber-500",
                border: "border-amber-500",
              },
              {
                id: "tax",
                label: "徴税",
                Icon: Coins,
                color: "text-yellow-500",
                glow: "bg-yellow-500",
                border: "border-yellow-500",
              },
              {
                id: "diplomacy",
                label: "外交",
                Icon: Handshake,
                color: "text-orange-500",
                glow: "bg-orange-500",
                border: "border-orange-500",
              },
              {
                id: "shop",
                label: "商店",
                Icon: Store,
                color: "text-purple-500",
                glow: "bg-purple-500",
                border: "border-purple-500",
              },
              {
                id: "item",
                label: "道具",
                Icon: Package,
                color: "text-emerald-500",
                glow: "bg-emerald-500",
                border: "border-emerald-500",
              },
              {
                id: "advisor",
                label: "軍師",
                Icon: Sparkles,
                color: "text-cyan-500",
                glow: "bg-cyan-500",
                border: "border-cyan-500",
              },
              {
                id: "navy",
                label: "水軍",
                Icon: Anchor,
                color: "text-cyan-400",
                glow: "bg-cyan-400",
                border: "border-cyan-400",
              },
              {
                id: "pvp",
                label: "対戦",
                Icon: Sword,
                color: "text-red-400",
                glow: "bg-red-400",
                border: "border-red-400",
              },
            ].map(({ id, label, Icon, color, glow, border }) => (
              <button
                key={id}
                onClick={() => setActiveTab(id)}
                className={`flex items-center gap-2 px-8 py-4 font-bold transition-all border-b-2 relative group ${activeTab === id ? `${color} ${border} bg-white/5` : "text-stone-500 border-transparent hover:text-stone-300"}`}
              >
                <Icon
                  className={`w-4 h-4 transition-transform group-hover:scale-110 ${activeTab === id ? color : ""}`}
                />
                <span className="text-xs tracking-[0.2em]">{label}</span>
                {activeTab === id && (
                  <motion.div
                    layoutId="tab-underline"
                    className={`absolute bottom-0 left-0 right-0 h-0.5 ${glow} shadow-[0_0_15px_rgba(0,0,0,0.5)]`}
                    style={{
                      filter: `drop-shadow(0 0 8px ${id === "map" ? "#ef4444" : id === "castles" ? "#a8a29e" : id === "deck" ? "#3b82f6" : id === "gacha" ? "#f59e0b" : id === "tax" ? "#eab308" : id === "diplomacy" ? "#f97316" : id === "shop" ? "#a855f7" : id === "item" ? "#10b981" : id === "navy" ? "#22d3ee" : id === "pvp" ? "#f87171" : "#06b6d4"})`,
                    }}
                  />
                )}
              </button>
            ))}
          </nav>

          <AnimatePresence mode="wait">
            {activeTab === "map" && (
              <motion.div
                key="map"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col relative overflow-hidden bg-dark-bg rounded-sm border border-white/5"
              >
                <div
                  ref={mapContainerRef}
                  className={`flex-1 overflow-auto relative custom-scrollbar ${isDragging ? "cursor-grabbing" : "cursor-grab"}`}
                  onMouseDown={handleMapMouseDown}
                  onMouseMove={handleMapMouseMove}
                  onMouseUp={() => setIsDragging(false)}
                >
                  <div
                    style={{
                      width: `${12800 * mapScale}px`,
                      height: `${9600 * mapScale}px`,
                    }}
                    className="relative bg-[#0F0F0F]"
                  >
                    <div
                      className="absolute inset-0 opacity-10"
                      style={{
                        backgroundImage:
                          "radial-gradient(#C5A059 1px, transparent 1px)",
                        backgroundSize: "120px 120px",
                      }}
                    ></div>

                    <svg
                      viewBox="0 0 100 100"
                      className="absolute inset-0 w-full h-full pointer-events-none"
                      preserveAspectRatio="none"
                    >
                      {/* 本州 */}
                      <path
                        d="M23,68 L28,64 L32,62 L40,60 L47,58 L50,55 L53,50 L55,54 L54,56 L58,50 L63,45 L68,35 L73,20 L75,18 L76,21 L78,19 L80,22 L80,25 L78,35 L76,40 L74,45 L72,50 L70,55 L72,62 L68,63 L66,60 L65,64 L64,62 L60,63 L57,65 L55,68 L55,74 L53,78 L50,81 L47,80 L44,76 L43,70 L40,69 L35,69 L30,70 L25,70 Z"
                        fill="#2d2924"
                        stroke="#4a4235"
                        strokeWidth="0.25"
                        strokeLinejoin="round"
                      />

                      {/* 九州 */}
                      <path
                        d="M19,72 L22,73 L24,75 L22,76 L24,79 L23,85 L20,87 L19,85 L18,80 L14,78 L15,75 L17,73 Z"
                        fill="#2d2924"
                        stroke="#4a4235"
                        strokeWidth="0.25"
                        strokeLinejoin="round"
                      />

                      {/* 四国 */}
                      <path
                        d="M35,74 L40,73 L42,75 L43,78 L41,82 L39,81 L34,83 L32,80 L33,76 Z"
                        fill="#2d2924"
                        stroke="#4a4235"
                        strokeWidth="0.25"
                        strokeLinejoin="round"
                      />

                      {/* 北海道 */}
                      <path
                        d="M76,16 L78,13 L81,14 L85,15 L88,16 L92,14 L95,10 L92,8 L94,5 L90,4 L85,6 L80,9 L76,11 Z"
                        fill="#2d2924"
                        stroke="#4a4235"
                        strokeWidth="0.25"
                        strokeLinejoin="round"
                      />

                      {/* 海上航路は水軍専用ページ（別頁）で管理するためメインマップからは非表示 */}

                      {ROUTE_CONNECTIONS.map(([a, b], idx) => {
                        const l1 = lands.find((l) => l.id === a);
                        const l2 = lands.find((l) => l.id === b);
                        if (!l1 || !l2) return null;

                        const isL1Alliance =
                          diplomacyRelations[l1.owner] === "alliance";
                        const isL2Alliance =
                          diplomacyRelations[l2.owner] === "alliance";
                        const isAllianceRoute = isL1Alliance || isL2Alliance;

                        const occupied =
                          occupiedLands.includes(a) ||
                          occupiedLands.includes(b);
                        const isMajorHighway = a < 100 && b < 100; // 本城同士の幹線

                        let strokeColor = "#332d25";
                        let strokeWidth = "0.12";
                        let dashArray = "0.8, 0.8";

                        if (isMajorHighway) {
                          // 幹線
                          strokeColor = occupied ? "#f59e0b" : "#78716c";
                          strokeWidth = "0.32";
                          dashArray = undefined; // 幹線は力強い実線

                          if (isAllianceRoute) {
                            strokeColor = "#10b981"; // 同盟幹線
                            strokeWidth = "0.4";
                          }
                        } else {
                          // 支線
                          strokeColor = occupied ? "#d97706" : "#443c30";
                          strokeWidth = "0.14";
                          dashArray = "0.4, 0.4"; // 繊細な点線

                          if (isAllianceRoute) {
                            strokeColor = "#059669";
                            strokeWidth = "0.2";
                          }
                        }

                        return (
                          <line
                            key={idx}
                            x1={l1.x}
                            y1={l1.y}
                            x2={l2.x}
                            y2={l2.y}
                            stroke={strokeColor}
                            strokeWidth={strokeWidth}
                            strokeDasharray={dashArray}
                          />
                        );
                      })}
                    </svg>

                    {lands.map((land) => {
                      const isOccupied = occupiedLands.includes(land.id);
                      const isSelected = selectedLand?.id === land.id;
                      const isAlliance =
                        diplomacyRelations[land.owner] === "alliance";
                      const isReachableTarget = !isOccupied && !isAlliance && isReachable(land.id);

                      let style = isOccupied
                        ? PLAYER_COLOR
                        : FACTION_COLORS[land.owner] || FACTION_COLORS["群"];
                      if (!isOccupied && isAlliance) {
                        style = {
                          bg: "bg-emerald-950/80",
                          border: "border-emerald-400",
                          text: "text-emerald-100",
                          glow: "shadow-[0_0_25px_rgba(16,185,129,0.7)]",
                        };
                      }

                      return (
                        <div
                          key={land.id}
                          onClick={() => {
                            if (dragDistance < 10) setSelectedLand(land);
                          }}
                          className={`absolute w-12 h-12 md:w-20 md:h-20 -ml-6 -mt-6 md:-ml-10 md:-mt-10 flex items-center justify-center transition-all ${style.text} ${isSelected ? "scale-125 z-20" : "hover:scale-110 z-10"}`}
                          style={{ left: `${land.x}%`, top: `${land.y}%` }}
                        >
                          <div
                            className={`relative flex items-center justify-center w-full h-full rounded-full border-4 ${style.bg} ${style.border} ${isSelected ? "shadow-[0_0_20px_white]" : style.glow}`}
                          >
                            {land.level >= 7 ? (
                              <Castle className="w-8 h-8 md:w-12 md:h-12" />
                            ) : (
                              <Tent className="w-6 h-6 md:w-8 md:h-8" />
                            )}
                            {land.id === capitalLandId && (
                              <Crown className="absolute -top-6 w-6 h-6 text-amber-400" />
                            )}

                            {/* 進軍可能バッジ（同盟領経由を含む） */}
                            {isReachableTarget && (
                              <>
                                <div className="absolute inset-x-0 inset-y-0 rounded-full border-2 border-red-500 animate-pulse pointer-events-none scale-105" />
                                <span className="absolute -top-1.5 -left-1.5 flex h-5 w-5 z-20 items-center justify-center">
                                  <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-red-500/50"></span>
                                  <span className="relative inline-flex rounded-full h-5 w-5 bg-red-700 border border-red-500 text-white shadow-md items-center justify-center">
                                    <Sword className="w-3 h-3 text-white" />
                                  </span>
                                </span>
                              </>
                            )}

                            {/* 港バッジ */}
                            {land.isPort && (
                              <div className="absolute -bottom-1 -right-1 bg-cyan-950 border border-cyan-400/80 p-0.5 md:p-1 rounded-full text-cyan-300 shadow-[0_0_12px_rgba(34,211,238,0.85)] z-10 animate-pulse">
                                <Anchor className="w-2.5 h-2.5 md:w-3.5 md:h-3.5" />
                              </div>
                            )}
                          </div>
                          <div className="absolute top-full mt-2 bg-black/80 px-2 py-0.5 rounded text-[10px] md:text-xs font-bold whitespace-nowrap border border-stone-800">
                            {land.name.split(" ")[0]}
                          </div>
                        </div>
                      );
                    })}
                  </div>
                </div>

                {/* 浮動地図ズームコントロール */}
                <div className="absolute right-6 top-6 flex flex-col gap-2 z-20 bg-stone-900/90 border border-amber-900/40 p-2.5 rounded-sm shadow-xl backdrop-blur-md">
                  <div className="text-[10px] text-amber-500 font-serif tracking-widest text-center border-b border-white/5 pb-1.5 font-bold">
                    合戦図・拡大縮小
                  </div>
                  <button
                    onClick={() =>
                      setMapScale((prev) => Math.min(3.0, prev + 0.1))
                    }
                    className="p-1 px-3 hover:bg-stone-800 text-stone-200 hover:text-white transition-all flex items-center gap-2 text-[11px] font-bold"
                    type="button"
                  >
                    <ZoomIn className="w-4 h-4 text-amber-400" />
                    <span>拡大する</span>
                  </button>
                  <button
                    onClick={() =>
                      setMapScale((prev) => Math.max(0.15, prev - 0.1))
                    }
                    className="p-1 px-3 hover:bg-stone-800 text-stone-200 hover:text-white transition-all flex items-center gap-2 text-[11px] font-bold border-t border-stone-800/60 pt-1.5"
                    type="button"
                  >
                    <ZoomOut className="w-4 h-4 text-stone-400" />
                    <span>縮小する</span>
                  </button>
                  <div className="text-[9px] text-stone-500 font-mono text-center pt-1 border-t border-stone-800 flex flex-col">
                    <span>戦域縮尺</span>
                    <span className="text-amber-400/80 font-bold mt-0.5">
                      {(mapScale * 25).toFixed(0)}倍
                    </span>
                  </div>
                </div>

                <AnimatePresence>
                  {selectedLand && (
                    <motion.div
                      initial={{ y: 200 }}
                      animate={{ y: 0 }}
                      exit={{ y: 200 }}
                      className="absolute bottom-0 inset-x-0 bg-stone-900/95 border-t border-amber-900 p-6 shadow-2xl backdrop-blur-md z-30"
                    >
                      <div className="flex justify-between items-center max-w-4xl mx-auto">
                        <div>
                          <div className="flex items-center gap-3 mb-2">
                            <h3 className="text-3xl font-bold text-white">
                              {selectedLand.name}
                            </h3>
                            <span className="bg-stone-800 px-3 py-1 rounded-full text-xs font-bold border border-stone-700 text-stone-400">
                              {selectedLand.owner} 支配
                            </span>
                          </div>
                          <div className="flex items-center gap-4 text-stone-400">
                            <span className="flex items-center gap-1">
                              <Sword className="w-4 h-4" /> 兵力:{" "}
                              {selectedLand.power.toLocaleString()}
                            </span>
                            <span className="flex items-center gap-1">
                              <MapIcon className="w-4 h-4" /> 難易度:{" "}
                              {selectedLand.level}
                            </span>
                            {(() => {
                              const neighbors = lands.filter((l) => {
                                const isNeighbor = ROUTE_CONNECTIONS.some(
                                  ([a, b]) => {
                                    return (
                                      (a === selectedLand.id && b === l.id) ||
                                      (b === selectedLand.id && a === l.id)
                                    );
                                  },
                                );
                                return (
                                  isNeighbor &&
                                  diplomacyRelations[l.owner] === "alliance"
                                );
                              });
                              if (
                                neighbors.length > 0 &&
                                !occupiedLands.includes(selectedLand.id) &&
                                diplomacyRelations[selectedLand.owner] !== "alliance"
                              ) {
                                const isDirectNeighbor = occupiedLands.some(ownedId => {
                                  return ROUTE_CONNECTIONS.some(([a, b]) => {
                                    return (a === selectedLand.id && b === ownedId) || (b === selectedLand.id && a === ownedId);
                                  });
                                });

                                return (
                                  <div className="flex flex-col gap-1.5">
                                    <span className="flex items-center gap-1 text-emerald-400 font-bold bg-emerald-950/40 border border-emerald-500/20 px-2 py-0.5 rounded text-[10px] w-fit">
                                      <Sparkles className="w-3.5 h-3.5 animate-pulse" />{" "}
                                      {neighbors
                                        .map((n) => n.name.split(" ")[0])
                                        .join("・")}
                                      大名（同盟）から援軍可能
                                    </span>
                                    {!isDirectNeighbor && isReachable(selectedLand.id) && (
                                      <span className="flex items-center gap-1 text-cyan-400 font-bold bg-cyan-950/40 border border-cyan-500/20 px-2 py-0.5 rounded text-[10px] w-fit">
                                        <Sparkles className="w-3.5 h-3.5 animate-pulse" />{" "}
                                        同盟領内を通過して進軍ルートを確保中
                                      </span>
                                    )}
                                  </div>
                                );
                              }
                              return null;
                            })()}
                          </div>
                        </div>
                        <div className="flex items-center gap-4">
                          {occupiedLands.includes(selectedLand.id) && (
                            selectedLand.id === capitalLandId ? (
                              <div className="bg-amber-950/80 border border-amber-600/60 px-4 py-2 rounded-lg text-amber-400 font-bold text-sm flex items-center gap-2">
                                <Crown className="w-4 h-4 text-amber-400 animate-bounce" />
                                <span>我が居城</span>
                              </div>
                            ) : (
                              <button
                                onClick={() => {
                                  setCapitalLandId(selectedLand.id);
                                  addLog(`【居城に設定】我が領国【${selectedLand.name.split(" ")[0]}】を新たなる「居城（本城）」に定めました！`, "success");
                                }}
                                className="bg-amber-900/60 hover:bg-amber-800 border border-amber-600 hover:border-amber-400 text-amber-200 hover:text-white px-4 py-2.5 rounded-lg font-bold text-sm transition-all flex items-center gap-1.5 cursor-pointer"
                              >
                                <Crown className="w-4 h-4 text-amber-400" />
                                <span>居城に定める</span>
                              </button>
                            )
                          )}
                          <select
                            value={selectedDeckForBattle}
                            onChange={(e) =>
                              setSelectedDeckForBattle(Number(e.target.value))
                            }
                            className="bg-stone-800 border border-stone-700 px-4 py-2 rounded-lg text-white outline-none focus:border-amber-500"
                          >
                            <option value={0}>第1部隊</option>
                            <option value={1}>第2部隊</option>
                            <option value={2}>第3部隊</option>
                          </select>
                          <button
                            disabled={
                              !isReachable(selectedLand.id) ||
                              occupiedLands.includes(selectedLand.id) ||
                              diplomacyRelations[selectedLand.owner] === "alliance"
                            }
                            onClick={() =>
                              startBattle(selectedLand, selectedDeckForBattle)
                            }
                            className="bg-red-700 hover:bg-red-600 disabled:bg-stone-800 text-white px-8 py-3 rounded-lg font-bold text-lg shadow-lg transition-all cursor-pointer"
                          >
                            {occupiedLands.includes(selectedLand.id)
                              ? "占領済み"
                              : diplomacyRelations[selectedLand.owner] === "alliance"
                                ? "同盟盟約中"
                                : isReachable(selectedLand.id)
                                  ? "出陣する"
                                  : "進軍不可"}
                          </button>
                          <button
                            onClick={() => setSelectedLand(null)}
                            className="text-stone-500 hover:text-white p-2"
                          >
                            <X className="w-6 h-6" />
                          </button>
                        </div>
                      </div>
                    </motion.div>
                  )}
                </AnimatePresence>
              </motion.div>
            )}

            {activeTab === "castles" && (
              <motion.div
                key="castles"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col gap-6"
              >
                {/* 城郭台帳ヘッダー・要約KPI */}
                <div className="grid grid-cols-1 md:grid-cols-4 gap-4 shrink-0">
                  <div className="bg-card-bg border border-white/5 p-4 rounded-sm flex items-center gap-4">
                    <div className="w-12 h-12 rounded-full bg-stone-900/50 border border-stone-800 flex items-center justify-center text-stone-400">
                      <Castle className="w-6 h-6" />
                    </div>
                    <div>
                      <span className="text-[10px] text-stone-500 uppercase tracking-widest block font-sans">
                        総城郭数
                      </span>
                      <span className="text-xl font-serif font-bold text-white">
                        {lands.length} 拠点
                      </span>
                    </div>
                  </div>

                  <div className="bg-card-bg border border-white/5 p-4 rounded-sm flex items-center gap-4">
                    <div className="w-12 h-12 rounded-full bg-rose-950/20 border border-rose-900/50 flex items-center justify-center text-rose-500">
                      <Shield className="w-6 h-6 animate-pulse" />
                    </div>
                    <div>
                      <span className="text-[10px] text-stone-500 uppercase tracking-widest block font-sans">
                        我が支配領（城）
                      </span>
                      <span className="text-xl font-serif font-bold text-rose-400">
                        {occupiedLands.length} 拠点{" "}
                        <span className="text-xs text-stone-500 font-sans font-normal">
                          (
                          {(
                            (occupiedLands.length / lands.length) *
                            100
                          ).toFixed(1)}
                          %)
                        </span>
                      </span>
                    </div>
                  </div>

                  <div className="bg-card-bg border border-white/5 p-4 rounded-sm flex items-center gap-4">
                    <div className="w-12 h-12 rounded-full bg-cyan-950/20 border border-cyan-900/50 flex items-center justify-center text-cyan-400">
                      <Anchor className="w-6 h-6" />
                    </div>
                    <div>
                      <span className="text-[10px] text-stone-500 uppercase tracking-widest block font-sans">
                        港湾拠点支配
                      </span>
                      <span className="text-xl font-serif font-bold text-cyan-400">
                        {
                          lands.filter(
                            (l) => l.isPort && occupiedLands.includes(l.id),
                          ).length
                        }{" "}
                        / {lands.filter((l) => l.isPort).length} 港
                      </span>
                    </div>
                  </div>

                  <div className="bg-card-bg border border-white/5 p-4 rounded-sm flex items-center gap-4">
                    <div className="w-12 h-12 rounded-full bg-amber-950/20 border border-amber-900/50 flex items-center justify-center text-amber-500">
                      <Users className="w-6 h-6" />
                    </div>
                    <div>
                      <span className="text-[10px] text-stone-500 uppercase tracking-widest block font-sans">
                        我が警備総兵力
                      </span>
                      <span className="text-xl font-mono font-bold text-amber-500">
                        {lands
                          .filter((l) => occupiedLands.includes(l.id))
                          .reduce((sum, l) => sum + (l.power || 0), 0)
                          .toLocaleString()}{" "}
                        兵
                      </span>
                    </div>
                  </div>
                </div>

                {/* フィルター・操作エリア */}
                <div className="bg-card-bg border border-white/5 p-4 rounded-sm shrink-0 flex flex-col md:flex-row justify-between items-center gap-4">
                  <div className="flex gap-2 w-full md:w-auto overflow-x-auto">
                    {[
                      { id: "all", label: "すべての城" },
                      { id: "mine", label: "我が支配下" },
                      { id: "attackable", label: "進軍可能（敵領）" },
                      { id: "others", label: "他国領・同盟領" },
                    ].map((tab) => (
                      <button
                        key={tab.id}
                        onClick={() => setCastleFilter(tab.id as any)}
                        className={`px-6 py-2 border text-xs font-bold tracking-widest transition-all cursor-pointer ${
                          castleFilter === tab.id
                            ? "bg-gold border-gold text-black shadow-lg shadow-gold/20"
                            : "bg-stone-900 border-stone-800 text-stone-400 hover:text-white hover:border-stone-500"
                        }`}
                      >
                        {tab.label}
                      </button>
                    ))}
                  </div>

                  <div className="text-right text-[11px] text-stone-500 italic font-serif">
                    「この台帳を以て、日ノ本の諸城の勢力と防備状況が俯瞰できまする。」
                  </div>
                </div>

                {/* 城郭リスト */}
                <div className="flex-1 overflow-y-auto custom-scrollbar bg-dark-bg border border-white/5 rounded-sm p-4">
                  <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                    {lands
                      .filter((land) => {
                        const isMine = occupiedLands.includes(land.id);
                        const isAlliance =
                          diplomacyRelations[land.owner] === "alliance";
                        const attackable = isReachable(land.id) && !isMine;

                        if (castleFilter === "mine") return isMine;
                        if (castleFilter === "attackable") return attackable;
                        if (castleFilter === "others")
                          return !isMine && !attackable;
                        return true;
                      })
                      .map((land) => {
                        const isMine = occupiedLands.includes(land.id);
                        const isAlliance =
                          diplomacyRelations[land.owner] === "alliance";
                        const isGoal = isReachable(land.id);
                        const isAttackable = isGoal && !isMine && !isAlliance;

                        return (
                          <div
                            key={land.id}
                            className={`bg-card-bg border rounded-sm p-5 flex flex-col transition-all relative overflow-hidden group hover:translate-y-[-2px] ${
                              isMine
                                ? "border-rose-950 hover:border-rose-800 shadow-[0_4px_20px_rgba(225,29,72,0.1)]"
                                : isAlliance
                                  ? "border-emerald-950/60 hover:border-emerald-800"
                                  : isAttackable
                                    ? "border-red-950 hover:border-red-800 hover:shadow-[0_4px_25px_rgba(185,28,28,0.15)]"
                                    : "border-stone-900 opacity-80"
                            }`}
                          >
                            {/* 右上の家紋デコレーション */}
                            <div className="absolute top-4 right-4 opacity-10 group-hover:opacity-25 transition-opacity">
                              <FactionKamon
                                faction={isMine ? "織田" : land.owner}
                                className="w-12 h-12 text-stone-400"
                              />
                            </div>

                            <div className="flex items-start gap-3 mb-4">
                              <span
                                className={`px-2 py-0.5 rounded-sm text-[10px] font-mono font-bold tracking-tighter ${
                                  isMine
                                    ? "bg-rose-950 text-rose-300 border border-rose-800"
                                    : isAlliance
                                      ? "bg-emerald-950 text-emerald-300 border border-emerald-800"
                                      : isAttackable
                                        ? "bg-red-950 text-red-300 border border-red-800"
                                        : "bg-stone-900 text-stone-500 border border-stone-800"
                                }`}
                              >
                                {isMine
                                  ? "我が領"
                                  : isAlliance
                                    ? "同盟"
                                    : isAttackable
                                      ? "進軍可能"
                                      : "進軍不可"}
                              </span>

                              {land.isPort && (
                                <span className="flex items-center gap-1 bg-cyan-950 text-cyan-400 border border-cyan-800 rounded-sm px-2 py-0.5 text-[9px] font-bold">
                                  <Anchor className="w-3 h-3" />
                                  <span>港湾都市</span>
                                </span>
                              )}
                            </div>

                            <h3 className="text-xl font-serif font-bold text-white mb-1 tracking-wider leading-none flex items-center gap-2">
                              <span>{land.name.split(" ")[0]}</span>
                              <span className="text-stone-500 text-xs font-sans font-normal">
                                {land.name.split(" ")[1] || ""}
                              </span>
                              {land.id === capitalLandId && (
                                <span className="bg-amber-950/60 border border-amber-500/50 text-amber-400 px-2 py-0.5 rounded text-[9px] font-bold flex items-center gap-1 animate-pulse">
                                  <Crown className="w-2.5 h-2.5 text-amber-400" />
                                  <span>居城</span>
                                </span>
                              )}
                            </h3>

                            <div className="flex items-center gap-4 text-[11px] text-stone-400 mb-4 border-b border-white/5 pb-3">
                              <span>
                                大名:{" "}
                                <strong className="font-serif text-stone-200">
                                  {isMine ? "貴殿" : `${land.owner}家`}
                                </strong>
                              </span>
                              <span>
                                城郭規模:{" "}
                                <strong className="font-mono text-gold">
                                  ★ {land.level}
                                </strong>
                              </span>
                            </div>

                            {/* データ詳細 */}
                            <div className="grid grid-cols-2 gap-3 mb-5 font-sans">
                              <div className="bg-black/20 p-2 rounded-sm border border-white/5 text-center">
                                <span className="text-[9px] text-stone-500 uppercase block tracking-widest">
                                  防御・守備兵数
                                </span>
                                <span
                                  className={`text-sm font-mono font-bold ${isMine ? "text-amber-500" : "text-stone-300"}`}
                                >
                                  {land.power.toLocaleString()} 兵
                                </span>
                              </div>
                              <div className="bg-black/20 p-2 rounded-sm border border-white/5 text-center">
                                <span className="text-[9px] text-stone-500 uppercase block tracking-widest">
                                  占領報酬(木/銭)
                                </span>
                                <span className="text-sm font-mono font-bold text-orange-400">
                                  {land.reward.toLocaleString()} 枚
                                </span>
                              </div>
                            </div>

                            {/* クイックアクション部 */}
                            <div className="mt-auto pt-2">
                              {isMine ? (
                                <div className="flex flex-col gap-2">
                                  <div className="flex gap-2">
                                    <button
                                      onClick={() => {
                                        setActiveTab("tax");
                                        addLog(
                                          `城郭【${land.name.split(" ")[0]}】から内政・民政を推進いたします。`,
                                          "info",
                                        );
                                      }}
                                      className="flex-1 py-2 bg-stone-900 hover:bg-stone-800 text-stone-300 hover:text-white rounded-sm border border-stone-800 hover:border-stone-600 transition-all font-serif font-bold text-xs tracking-wider cursor-pointer"
                                    >
                                      内政へ飛ぶ
                                    </button>
                                    {land.isPort && (
                                      <button
                                        onClick={() => {
                                          setActiveTab("navy");
                                          addLog(
                                            `【水軍】港湾都市【${land.name.split(" ")[0]}】の造船施設へと出向します。`,
                                            "info",
                                          );
                                        }}
                                        className="flex-1 py-2 bg-cyan-950 hover:bg-cyan-900 border border-cyan-800 text-cyan-400 hover:text-cyan-200 rounded-sm transition-all font-serif font-bold text-xs tracking-wider flex items-center justify-center gap-1 cursor-pointer"
                                      >
                                        <Anchor className="w-3.5 h-3.5" />
                                        水軍へ飛ぶ
                                      </button>
                                    )}
                                  </div>

                                  {land.id === capitalLandId ? (
                                    <div className="w-full py-1.5 bg-amber-950/20 border border-amber-900/30 rounded-sm text-center text-[10px] text-amber-400 font-bold flex items-center justify-center gap-1">
                                      <Crown className="w-3 h-3 text-amber-500 animate-pulse" />
                                      <span>現在、日ノ本我が支配領の「居城」に指定中</span>
                                    </div>
                                  ) : (
                                    <button
                                      onClick={() => {
                                        setCapitalLandId(land.id);
                                        addLog(
                                          `【居城を設定】我が領国【${land.name.split(" ")[0]}】を新たなる「居城（本城）」に定めました！`,
                                          "success",
                                        );
                                      }}
                                      className="w-full py-1.5 bg-amber-950/10 hover:bg-amber-900/25 border border-amber-800/30 hover:border-amber-600 rounded-sm text-amber-400 hover:text-amber-200 font-serif font-bold text-xs tracking-wider transition-all flex items-center justify-center gap-1.5 cursor-pointer"
                                    >
                                      <Crown className="w-3.5 h-3.5 text-amber-500" />
                                      居城に定める
                                    </button>
                                  )}
                                </div>
                              ) : isAlliance ? (
                                <div className="p-2 border border-emerald-900/30 bg-emerald-950/10 rounded-sm text-center">
                                  <span className="text-[10px] text-emerald-400 font-serif">
                                    同盟同盟の下、不可侵の平和が保たれています。
                                  </span>
                                </div>
                              ) : isAttackable ? (
                                <div className="flex flex-col gap-2 bg-black/30 border border-red-900/20 p-3 rounded-sm">
                                  <div className="flex items-center justify-between text-[11px]">
                                    <span className="text-stone-500 font-sans">
                                      出撃部隊:
                                    </span>
                                    <select
                                      id={`deck-select-${land.id}`}
                                      className="bg-stone-900 border border-stone-800 rounded-sm text-stone-300 text-[10px] px-2 py-0.5 outline-none focus:border-red-500 cursor-pointer font-sans"
                                    >
                                      <option value={0}>
                                        第1部隊 (戦力:{" "}
                                        {calculateDeckPower(0).toLocaleString()}
                                        )
                                      </option>
                                      <option value={1}>
                                        第2部隊 (戦力:{" "}
                                        {calculateDeckPower(1).toLocaleString()}
                                        )
                                      </option>
                                      <option value={2}>
                                        第3部隊 (戦力:{" "}
                                        {calculateDeckPower(2).toLocaleString()}
                                        )
                                      </option>
                                    </select>
                                  </div>
                                  <button
                                    onClick={() => {
                                      const select = document.getElementById(
                                        `deck-select-${land.id}`,
                                      ) as HTMLSelectElement;
                                      const idx = select
                                        ? Number(select.value)
                                        : 0;
                                      startBattle(land, idx);
                                    }}
                                    className="w-full py-2 bg-red-800 hover:bg-red-700 text-white rounded-sm border border-red-700 hover:border-red-600 transition-all font-serif font-bold text-xs tracking-widest block text-center cursor-pointer"
                                  >
                                    この城へ出陣する！
                                  </button>
                                </div>
                              ) : (
                                <div className="p-2 border border-stone-900 bg-stone-950/20 rounded-sm text-center opacity-55">
                                  <span className="text-[10px] text-stone-600">
                                    周囲の街道を占領しておらず、進軍路がありませぬ。
                                  </span>
                                </div>
                              )}
                            </div>
                          </div>
                        );
                      })}
                  </div>
                </div>
              </motion.div>
            )}

            {activeTab === "deck" && (
              <motion.div
                key="deck"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col gap-6"
              >
                <div className="flex-1 bg-card-bg rounded-sm border border-white/5 p-8 flex flex-col">
                  <div className="flex gap-4 mb-8">
                    {[0, 1, 2].map((idx) => (
                      <button
                        key={idx}
                        onClick={() => setActiveDeckIndex(idx)}
                        className={`flex-1 py-4 font-bold transition-all border outline-none ${activeDeckIndex === idx ? "bg-white/5 border-gold text-gold shadow-[0_0_20px_rgba(197,160,89,0.1)]" : "bg-transparent border-white/5 text-stone-500 hover:text-stone-300"}`}
                      >
                        <span className="text-[10px] tracking-widest uppercase block opacity-50 mb-1">
                          第 {idx + 1} 部隊
                        </span>
                        <span className="text-sm">
                          戦力: {calculateDeckPower(idx).toLocaleString()}
                        </span>
                      </button>
                    ))}
                  </div>

                  <div className="grid grid-cols-5 gap-6 flex-1">
                    {decks[activeDeckIndex].map((id, idx) => {
                      const g = id
                        ? ownedGenerals.find((og) => og.uniqueId === id)
                        : null;
                      return (
                        <div
                          key={idx}
                          className="bg-dark-bg border border-white/5 p-6 flex flex-col items-center justify-center relative group"
                        >
                          <span className="absolute top-3 left-3 text-[9px] text-stone-600 uppercase font-bold tracking-tighter">
                            {idx === 0 ? "主将" : "副将"}
                          </span>
                          {g ? (
                            <div className="text-center w-full">
                              <div className="mb-3 flex justify-center">
                                <RarityStars count={g.rarity} />
                              </div>
                              <h4 className="font-serif font-bold text-lg text-white mb-1 group-hover:text-gold transition-colors">
                                {g.name}
                              </h4>
                              <p className="text-[10px] text-stone-500 mb-4 tracking-wider uppercase">
                                {g.faction} / レベル {g.level}
                              </p>
                              <div
                                className={`text-[9px] px-3 py-1 rounded-full inline-block mb-6 border border-white/10 uppercase tracking-widest bg-white/5`}
                              >
                                {g.troop.name}
                              </div>
                              <button
                                onClick={() => {
                                  const next = [...decks];
                                  next[activeDeckIndex][idx] = null;
                                  setDecks(next);
                                }}
                                className="w-full bg-transparent border border-red-900/30 text-red-700 text-[10px] py-2 rounded-sm hover:bg-red-900/10 transition-colors uppercase font-bold tracking-widest"
                              >
                                解任
                              </button>
                            </div>
                          ) : (
                            <div className="text-stone-800 flex flex-col items-center gap-3">
                              <Users className="w-10 h-10 opacity-10" />
                              <span className="text-[10px] tracking-widest uppercase font-bold">
                                空スロット
                              </span>
                            </div>
                          )}
                        </div>
                      );
                    })}
                  </div>
                </div>

                <div className="flex-1 bg-dark-bg border border-white/5 p-8 flex flex-col min-h-0">
                  <div className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 mb-8">
                    <div className="flex items-center gap-4">
                      <h3 className="font-serif font-bold text-2xl flex items-center gap-3 text-white tracking-widest">
                        <Users className="w-6 h-6 text-gold" /> 所持武将一覧
                      </h3>
                      <span className="text-[10px] bg-white/5 border border-white/10 px-2 py-0.5 rounded text-stone-500 font-mono">
                        {ownedGenerals.length} 名
                      </span>
                    </div>

                    <div className="flex flex-wrap gap-2">
                      <select
                        value={generalFilter}
                        onChange={(e) => setGeneralFilter(e.target.value)}
                        className="bg-white/5 border border-white/10 px-3 py-1.5 rounded text-[10px] text-stone-300 font-bold tracking-widest outline-none focus:border-gold"
                      >
                        <option value="ALL">全勢力</option>
                        {Object.values(FACTIONS).map((f) => (
                          <option key={f as string} value={f as string}>
                            {f as string}
                          </option>
                        ))}
                      </select>

                      <select
                        value={generalSort}
                        onChange={(e) => setGeneralSort(e.target.value)}
                        className="bg-white/5 border border-white/10 px-3 py-1.5 rounded text-[10px] text-stone-300 font-bold tracking-widest outline-none focus:border-gold"
                      >
                        <option value="RARITY">希少度順</option>
                        <option value="LEVEL">レベル順</option>
                        <option value="NAME">名前順</option>
                      </select>
                    </div>
                  </div>

                  <div className="flex-1 overflow-y-auto pr-2 custom-scrollbar">
                    {ownedGenerals.filter(
                      (g) =>
                        generalFilter === "ALL" || g.faction === generalFilter,
                    ).length > 0 ? (
                      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 gap-4">
                        {ownedGenerals
                          .filter(
                            (g) =>
                              generalFilter === "ALL" ||
                              g.faction === generalFilter,
                          )
                          .sort((a, b) => {
                            if (generalSort === "RARITY")
                              return b.rarity - a.rarity;
                            if (generalSort === "LEVEL")
                              return b.level - a.level;
                            if (generalSort === "NAME")
                              return a.name.localeCompare(b.name);
                            return 0;
                          })
                          .map((g) => {
                            const equipped = isGeneralEquipped(g.uniqueId);
                            const fStyle =
                              FACTION_COLORS[g.faction] || FACTION_COLORS["群"];

                            return (
                              <div
                                key={g.uniqueId}
                                className={`group bg-white/5 border ${equipped ? "border-white/10 opacity-40" : "border-white/10 hover:border-gold/50"} rounded px-4 py-4 transition-all relative flex flex-col`}
                              >
                                <div className="flex justify-between items-start mb-3">
                                  <div
                                    className={`text-[8px] px-2 py-0.5 rounded border ${fStyle.border} ${fStyle.text} font-bold tracking-tighter`}
                                  >
                                    {g.faction}
                                  </div>
                                  <RarityStars count={g.rarity} />
                                </div>

                                <div className="flex gap-3 mb-4">
                                  <div className="w-12 h-12 bg-white/5 rounded-sm flex items-center justify-center border border-white/10 shrink-0">
                                    <Users
                                      className={`w-6 h-6 ${equipped ? "text-stone-700" : "text-gold"}`}
                                    />
                                  </div>
                                  <div className="min-w-0">
                                    <h4 className="font-serif font-bold text-white text-sm truncate">
                                      {g.name}
                                    </h4>
                                    <p className="text-[10px] text-stone-500 font-mono tracking-tighter">
                                      LV.{g.level} / {g.troop.name}
                                    </p>
                                  </div>
                                </div>

                                <div className="grid grid-cols-3 gap-1 mb-4 text-[9px] font-bold tracking-tighter text-center">
                                  <div className="bg-white/5 py-1 rounded border border-white/5">
                                    <div className="text-stone-500 mb-0.5">
                                      武力
                                    </div>
                                    <div className="text-red-400">{g.atk}</div>
                                  </div>
                                  <div className="bg-white/5 py-1 rounded border border-white/5">
                                    <div className="text-stone-500 mb-0.5">
                                      知略
                                    </div>
                                    <div className="text-blue-400">{g.int}</div>
                                  </div>
                                  <div className="bg-white/5 py-1 rounded border border-white/5">
                                    <div className="text-stone-500 mb-0.5">
                                      統率
                                    </div>
                                    <div className="text-green-400">
                                      {g.def}
                                    </div>
                                  </div>
                                </div>

                                <div className="mb-3 text-[9px] text-stone-500 font-serif leading-tight">
                                  <span className="text-gold opacity-50 mr-2">
                                    秘技:
                                  </span>
                                  {g.skill}
                                </div>

                                {equippedWeapons[g.uniqueId] ? (() => {
                                  const wpObj = MASTER_WEAPONS.find((w) => w.id === equippedWeapons[g.uniqueId]);
                                  return (
                                    <div className="mb-3 bg-amber-950/15 border border-amber-500/25 px-2 py-1 rounded text-[9px] text-amber-300 font-sans flex justify-between items-center shadow-inner">
                                      <div className="flex items-center gap-1.5">
                                        <span className="text-[11px]">⚔️</span>
                                        <span className="font-bold">{wpObj?.name}</span>
                                      </div>
                                      <div className="flex gap-1.5 text-[8px] opacity-75 font-mono">
                                        {wpObj?.atk !== 0 && <span className="text-red-400">武+{wpObj?.atk}</span>}
                                        {wpObj?.int !== 0 && <span className="text-blue-400">知+{wpObj?.int}</span>}
                                        {wpObj?.def !== 0 && <span className="text-green-400">統+{wpObj?.def}</span>}
                                      </div>
                                    </div>
                                  );
                                })() : (
                                  <div className="mb-3 bg-stone-900/30 border border-stone-800 px-2 py-1 rounded text-[9px] text-stone-600 font-sans italic text-center">
                                    兵装未装備
                                  </div>
                                )}

                                <div className="grid grid-cols-2 gap-2 mt-auto">
                                  <button
                                    onClick={() => setTrainingGeneral(g)}
                                    className="bg-amber-950/20 border border-amber-500/20 hover:bg-amber-500 hover:text-black hover:border-amber-500 py-2 rounded-sm text-[10px] font-bold tracking-widest text-amber-300 transition-all font-sans"
                                  >
                                    養成・兵装 ⚙️
                                  </button>

                                  <button
                                    disabled={equipped}
                                    onClick={() => {
                                      const next = [...decks];
                                      const slot =
                                        next[activeDeckIndex].indexOf(null);
                                      if (slot !== -1) {
                                        next[activeDeckIndex][slot] = g.uniqueId;
                                        setDecks(next);
                                      } else {
                                        addLog(
                                          "部隊に空きスロットがありません。",
                                          "error",
                                        );
                                      }
                                    }}
                                    className={`py-2 rounded-sm text-[10px] font-bold tracking-widest transition-all ${equipped ? "bg-stone-800 text-stone-600 border border-stone-700 cursor-not-allowed" : "bg-white/5 border border-white/10 text-stone-300 hover:bg-gold hover:text-black hover:border-gold"}`}
                                  >
                                    {equipped ? "配属中" : "部隊配属"}
                                  </button>
                                </div>
                              </div>
                            );
                          })}
                      </div>
                    ) : (
                      <div className="h-full flex flex-col items-center justify-center text-stone-800 opacity-20">
                        <Users className="w-16 h-16 mb-4" />
                        <span className="font-serif tracking-[0.3em] uppercase">
                          該当する武将はいません
                        </span>
                      </div>
                    )}
                  </div>
                </div>
              </motion.div>
            )}

            {activeTab === "gacha" && (
              <motion.div
                key="gacha"
                className="h-full flex flex-col items-center justify-center space-y-12"
              >
                <div className="text-center">
                  <h2 className="text-6xl font-serif font-bold text-gold mb-6 tracking-[0.4em] uppercase">
                    武将登用
                  </h2>
                  <p className="text-stone-500 font-sans tracking-[0.2em] font-light opacity-60">
                    乱世に名を馳せる英雄を、貴殿の配下に招き入れよ。
                  </p>
                </div>
                <div className="flex gap-12">
                  <button
                    onClick={() => drawGacha(1)}
                    className="group bg-card-bg border border-white/5 p-12 rounded-sm hover:border-gold hover:shadow-[0_0_50px_rgba(197,160,89,0.1)] transition-all text-center min-w-[280px]"
                  >
                    <span className="block text-[10px] tracking-[0.3em] uppercase text-stone-500 mb-4 group-hover:text-gold transition-colors">
                      単発
                    </span>
                    <span className="block text-2xl font-serif text-white mb-8">
                      登用・壱
                    </span>
                    <div className="flex items-center justify-center gap-3 text-gold">
                      <Crown className="w-5 h-5 opacity-50" />{" "}
                      <span className="font-mono text-xl mr-1">200</span>
                    </div>
                  </button>
                  <button
                    onClick={() => drawGacha(10)}
                    className="group bg-white/5 border border-gold/30 p-12 rounded-sm hover:border-gold hover:shadow-[0_0_60px_rgba(197,160,89,0.2)] transition-all text-center min-w-[280px] relative overflow-hidden"
                  >
                    <div className="absolute top-0 right-0 bg-gold text-black text-[9px] font-bold px-4 py-1 tracking-widest">
                      忠義
                    </div>
                    <span className="block text-[10px] tracking-[0.3em] uppercase text-gold/50 mb-4">
                      十連
                    </span>
                    <span className="block text-2xl font-serif text-white mb-8">
                      登用・拾
                    </span>
                    <div className="flex items-center justify-center gap-3 text-gold">
                      <Crown className="w-5 h-5" />{" "}
                      <span className="font-mono text-xl mr-1">1900</span>
                    </div>
                  </button>
                </div>
              </motion.div>
            )}

            {activeTab === "tax" && (
              <motion.div
                key="tax"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col p-8 bg-dark-bg"
              >
                <div className="mb-8">
                  <h2 className="text-3xl font-serif font-bold text-gold tracking-widest uppercase flex items-center gap-4">
                    <Coins className="w-8 h-8 opacity-50" /> 徴税と民政
                  </h2>
                  <p className="text-stone-500 font-sans mt-2 opacity-60">
                    支配下の村落や農民から年貢を徴収し、戦費を調達する。寛大な治世を敷くか、過酷な取り立てを行うかは貴殿次第。
                  </p>
                </div>

                <div className="grid grid-cols-1 lg:grid-cols-3 gap-8 flex-1 overflow-y-auto custom-scrollbar pb-4 animate-fade-in">
                  {/* 左カラム: 民忠・財政状況 */}
                  <div className="bg-card-bg border border-white/5 p-6 rounded-sm flex flex-col space-y-6">
                    <h3 className="font-serif font-bold text-lg text-white border-b border-white/5 pb-3">
                      民政・支配状況
                    </h3>

                    {/* 民忠 (People's Loyalty) ステータス */}
                    <div>
                      <div className="flex justify-between items-center text-xs font-bold mb-2">
                        <span className="text-stone-400">
                          農民の支持度（民忠）
                        </span>
                        <span
                          className={`font-mono text-base ${
                            peoplesLoyalty >= 80
                              ? "text-emerald-400"
                              : peoplesLoyalty >= 50
                                ? "text-yellow-400"
                                : peoplesLoyalty >= 25
                                  ? "text-orange-400"
                                  : "text-red-500 animate-pulse font-extrabold"
                          }`}
                        >
                          {peoplesLoyalty} / 100
                        </span>
                      </div>

                      <div className="w-full bg-black/50 h-3 rounded-full border border-white/10 overflow-hidden">
                        <div
                          className={`h-full transition-all duration-500 ${
                            peoplesLoyalty >= 80
                              ? "bg-gradient-to-r from-emerald-600 to-emerald-400"
                              : peoplesLoyalty >= 50
                                ? "bg-gradient-to-r from-yellow-600 to-yellow-400"
                                : peoplesLoyalty >= 25
                                  ? "bg-gradient-to-r from-orange-600 to-orange-400"
                                  : "bg-gradient-to-r from-red-700 to-red-500 animate-pulse"
                          }`}
                          style={{ width: `${peoplesLoyalty}%` }}
                        />
                      </div>

                      <p
                        className={`text-[11px] font-medium leading-relaxed mt-3 p-3 rounded bg-white/5 border border-white/5 ${
                          peoplesLoyalty >= 80
                            ? "text-emerald-300/80 border-emerald-500/10"
                            : peoplesLoyalty >= 50
                              ? "text-yellow-300/80 border-yellow-500/10"
                              : peoplesLoyalty >= 25
                                ? "text-orange-300/80 border-orange-500/10"
                                : "text-red-300 border-red-500/20"
                        }`}
                      >
                        {peoplesLoyalty >= 80
                          ? "【極楽平穏】人々は殿の温情に感謝し、日々耕作に励んでいます。一揆の気配はありません。"
                          : peoplesLoyalty >= 50
                            ? "【承服】生活は楽ではありませんが、人々は殿の命令に静かに従っています。"
                            : peoplesLoyalty >= 25
                              ? "【怨嗟警戒】重い負担により、人々は苦しんでいます。一揆の危険性が上昇しています！"
                              : "【暴動寸前】過酷な治世への怒りが頂点に達しています！徴税を行えば、ただちに一揆が引き起こされるでしょう。"}
                      </p>
                    </div>

                    {/* 財政情報 */}
                    <div className="space-y-3 pt-4 border-t border-white/5 font-sans">
                      <div className="flex justify-between items-center text-xs">
                        <span className="text-stone-400">支配国の総数</span>
                        <span className="font-mono text-white font-bold">
                          {occupiedLands.length} 箇所
                        </span>
                      </div>
                      <div className="flex justify-between items-center text-xs">
                        <span className="text-stone-400">農民登録人口</span>
                        <span className="font-mono text-white font-bold">
                          {(
                            Math.max(occupiedLands.length, 1) * 12.5
                          ).toLocaleString()}{" "}
                          千人
                        </span>
                      </div>
                      <div className="flex justify-between items-center text-xs">
                        <span className="text-stone-400">
                          配下の農民国（農民所持数）
                        </span>
                        <span className="font-mono text-emerald-400 font-bold">
                          {
                            ownedGenerals.filter((g) => g.name === "農民")
                              .length
                          }{" "}
                          名
                        </span>
                      </div>
                      <div className="flex justify-between items-center text-xs border-t border-white/5 pt-2">
                        <span className="text-stone-400">
                          銅銭徴収規模 (農民数基準)
                        </span>
                        <span className="font-mono text-amber-500 font-bold">
                          ×{" "}
                          {
                            ownedGenerals.filter((g) => g.name === "農民")
                              .length
                          }
                        </span>
                      </div>
                      <div className="flex justify-between items-center text-xs">
                        <span className="text-stone-400">
                          小判徴収規模 (支配国数基準)
                        </span>
                        <span className="font-mono text-amber-500 font-bold">
                          × {Math.max(occupiedLands.length, 1)}
                        </span>
                      </div>

                      {/* 海港・海運恩恵表示 */}
                      {(lands.some(
                        (l) => l.isPort && occupiedLands.includes(l.id),
                      ) ||
                        (navyShips?.sengokubune || 0) > 0) && (
                        <div className="pt-3 border-t border-cyan-500/10 space-y-2 mt-2 bg-cyan-950/20 p-3 rounded border border-cyan-500/10 text-[11px]">
                          <div className="flex items-center gap-1.5 text-cyan-400 font-serif font-bold text-xs">
                            <Anchor className="w-3.5 h-3.5 text-cyan-400 animate-pulse" />{" "}
                            海港交易・海運物流効果
                          </div>
                          <div className="flex justify-between items-center">
                            <span className="text-stone-400">支配海港数</span>
                            <span className="font-mono text-cyan-300 font-bold">
                              {
                                lands.filter(
                                  (l) =>
                                    l.isPort && occupiedLands.includes(l.id),
                                ).length
                              }{" "}
                              港
                            </span>
                          </div>
                          {lands.filter(
                            (l) => l.isPort && occupiedLands.includes(l.id),
                          ).length > 0 && (
                            <div className="flex justify-between items-center pb-1">
                              <span className="text-stone-400">
                                交易増益加算値
                              </span>
                              <span className="font-mono text-emerald-400 font-bold">
                                小判 +
                                {(
                                  lands.filter(
                                    (l) =>
                                      l.isPort && occupiedLands.includes(l.id),
                                  ).length * 100
                                ).toLocaleString()}{" "}
                                / 銅銭 +
                                {(
                                  lands.filter(
                                    (l) =>
                                      l.isPort && occupiedLands.includes(l.id),
                                  ).length * 1200
                                ).toLocaleString()}
                              </span>
                            </div>
                          )}
                          <div className="flex justify-between items-center border-t border-cyan-500/10 pt-1.5">
                            <span className="text-stone-400">
                              物流ブースト (千石船 {navyShips?.sengokubune || 0}{" "}
                              隻)
                            </span>
                            <span className="font-mono text-cyan-300 font-bold">
                              +{((navyShips?.sengokubune || 0) * 10).toFixed(0)}
                              % 徴収ブースト
                            </span>
                          </div>
                        </div>
                      )}
                    </div>

                    {/* 施し（Give Alms）セクション */}
                    <div className="pt-6 border-t border-white/5 mt-auto">
                      <div className="mb-4">
                        <h4 className="text-xs font-bold text-white mb-1">
                          民政：施しを行う
                        </h4>
                        <p className="text-[10px] text-stone-500">
                          銅銭を投じて農民へ米や炊き出しを施し、民心の安定（民忠回復）を図ります。
                        </p>
                      </div>
                      <button
                        onClick={handleGiveAlms}
                        className="w-full bg-emerald-500/15 border border-emerald-500/30 hover:bg-emerald-500 hover:text-black hover:border-emerald-500 transition-all py-3 text-[11px] font-bold tracking-widest text-emerald-400 flex items-center justify-center gap-2 rounded-sm select-none"
                      >
                        <Sparkles className="w-3.5 h-3.5" /> 米を配給する (費用:
                        2,500 銅銭)
                      </button>
                    </div>
                  </div>

                  {/* 中・右カラム: 徴税方針の決定(3種類) */}
                  <div className="lg:col-span-2 flex flex-col space-y-4">
                    <div className="bg-card-bg border border-white/5 p-6 rounded-sm">
                      <h3 className="font-serif font-bold text-lg text-white mb-2">
                        年貢徴収の方針選択
                      </h3>
                      <p className="text-stone-500 text-xs leading-relaxed">
                        獲得できる銅銭は「配下の農民カード所持人数」に依存し、獲得できる小判は「支配領地の総数」に依存します。
                        <br />
                        登用（ガチャ）で農民を多く抱え込むほど、一度に手に入る銅銭が倍増します。
                      </p>
                    </div>

                    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                      {/* 寛大 */}
                      <div className="bg-card-bg border border-white/5 p-6 rounded-sm flex flex-col hover:border-white/10 transition-all">
                        <div className="px-2.5 py-1 self-start rounded bg-emerald-500/10 border border-emerald-500/20 text-emerald-400 text-[9px] font-bold uppercase tracking-widest mb-4">
                          寛大・三公七民
                        </div>
                        <h4 className="font-serif font-bold text-xl text-white mb-2">
                          寛大なる徴税
                        </h4>
                        <p className="text-stone-500 text-[11px] leading-relaxed mb-6">
                          農民の手元に多くを残し、信頼を得る施策。民忠が上昇し、一揆は絶対に起きません。
                        </p>

                        <div className="space-y-2 mb-8 bg-black/30 p-4 rounded-sm border border-white/5 text-xs font-mono font-sans">
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              獲得銅銭 (農民数)
                            </span>
                            <span className="text-stone-300">
                              +
                              {(
                                ownedGenerals.filter((g) => g.name === "農民")
                                  .length * 1000
                              ).toLocaleString()}
                            </span>
                          </div>
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              獲得小判 (領地数)
                            </span>
                            <span className="text-stone-300">
                              +
                              {(
                                Math.max(occupiedLands.length, 1) * 80
                              ).toLocaleString()}
                            </span>
                          </div>
                          <div className="flex justify-between border-t border-white/5 pt-2">
                            <span className="text-stone-500 font-sans">
                              民忠変化
                            </span>
                            <span className="text-emerald-400 font-bold">
                              +5
                            </span>
                          </div>
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              一揆発生率
                            </span>
                            <span className="text-emerald-400 font-bold">
                              0 %
                            </span>
                          </div>
                        </div>

                        <button
                          onClick={() => handleCollectTax("generous")}
                          className="w-full mt-auto bg-white/5 border border-white/10 py-3 text-[10px] font-bold tracking-widest text-stone-300 hover:bg-emerald-500 hover:text-black hover:border-emerald-500 transition-all rounded-sm flex items-center justify-center gap-1.5 select-none"
                        >
                          寛大なる年貢を命ず
                        </button>
                      </div>

                      {/* 標準 */}
                      <div className="bg-card-bg border border-white/5 p-6 rounded-sm flex flex-col hover:border-white/10 transition-all">
                        <div className="px-2.5 py-1 self-start rounded bg-yellow-500/10 border border-yellow-500/20 text-yellow-400 text-[9px] font-bold uppercase tracking-widest mb-4">
                          標準・五公五民
                        </div>
                        <h4 className="font-serif font-bold text-xl text-white mb-2">
                          標準の徴税
                        </h4>
                        <p className="text-stone-500 text-[11px] leading-relaxed mb-6">
                          天下の規範とされる中庸な税率。相応の収入が得られるが、負担により少し不満が蓄積します。
                        </p>

                        <div className="space-y-2 mb-8 bg-black/30 p-4 rounded-sm border border-white/5 text-xs font-mono font-sans">
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              獲得銅銭 (農民数)
                            </span>
                            <span className="text-stone-300">
                              +
                              {(
                                ownedGenerals.filter((g) => g.name === "農民")
                                  .length * 2500
                              ).toLocaleString()}
                            </span>
                          </div>
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              獲得小判 (領地数)
                            </span>
                            <span className="text-stone-300">
                              +
                              {(
                                Math.max(occupiedLands.length, 1) * 220
                              ).toLocaleString()}
                            </span>
                          </div>
                          <div className="flex justify-between border-t border-white/5 pt-2">
                            <span className="text-stone-500 font-sans">
                              民忠変化
                            </span>
                            <span className="text-orange-400 font-bold">
                              -15
                            </span>
                          </div>
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              一揆発生率
                            </span>
                            <span className="text-yellow-400 font-bold">
                              {peoplesLoyalty < 55 ? "32 %" : "12 %"}
                            </span>
                          </div>
                        </div>

                        <button
                          onClick={() => handleCollectTax("standard")}
                          className="w-full mt-auto bg-white/5 border border-white/10 py-3 text-[10px] font-bold tracking-widest text-stone-300 hover:bg-yellow-500 hover:text-black hover:border-yellow-500 transition-all rounded-sm flex items-center justify-center gap-1.5 select-none"
                        >
                          適正なる五公五民
                        </button>
                      </div>

                      {/* 苛烈 */}
                      <div className="bg-card-bg border border-red-500/10 p-6 rounded-sm flex flex-col hover:border-red-500/30 transition-all relative">
                        <div className="px-2.5 py-1 self-start rounded bg-red-500/10 border border-red-500/20 text-red-400 text-[9px] font-bold uppercase tracking-widest mb-4">
                          苛政・八公二民
                        </div>
                        <h4 className="font-serif font-bold text-xl text-white mb-2">
                          苛烈なる徴税
                        </h4>
                        <p className="text-stone-500 text-[11px] leading-relaxed mb-6 text-red-300/60">
                          非道なる年貢徴収。莫大なる富を直ちに調達する。人々の怨嗟は極まり、一揆が多発します。
                        </p>

                        <div className="space-y-2 mb-8 bg-black/40 p-4 rounded-sm border border-red-500/5 text-xs font-mono font-sans">
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              獲得銅銭 (農民数)
                            </span>
                            <span className="text-gold font-bold">
                              +
                              {(
                                ownedGenerals.filter((g) => g.name === "農民")
                                  .length * 6000
                              ).toLocaleString()}
                            </span>
                          </div>
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              獲得小判 (領地数)
                            </span>
                            <span className="text-gold font-bold">
                              +
                              {(
                                Math.max(occupiedLands.length, 1) * 550
                              ).toLocaleString()}
                            </span>
                          </div>
                          <div className="flex justify-between border-t border-white/5 pt-2">
                            <span className="text-stone-500 font-sans">
                              民忠変化
                            </span>
                            <span className="text-red-500 font-extrabold">
                              -35
                            </span>
                          </div>
                          <div className="flex justify-between">
                            <span className="text-stone-500 font-sans">
                              一揆発生率
                            </span>
                            <span className="text-red-400 font-extrabold">
                              {peoplesLoyalty < 55 ? "90 %" : "45 %"}
                            </span>
                          </div>
                        </div>

                        <button
                          onClick={() => handleCollectTax("harsh")}
                          className="w-full mt-auto bg-red-500/10 border border-red-500/30 py-3 text-[10px] font-bold tracking-widest text-red-400 hover:bg-red-500 hover:text-black hover:border-red-500 transition-all rounded-sm flex items-center justify-center gap-1.5 select-none"
                        >
                          不退転の過酷なる徴税
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </motion.div>
            )}

            {activeTab === "diplomacy" && (
              <motion.div
                key="diplomacy"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col p-8"
              >
                <div className="mb-8">
                  <h2 className="text-3xl font-serif font-bold text-gold tracking-widest uppercase flex items-center gap-4">
                    <Handshake className="w-8 h-8 opacity-50" /> 外交
                  </h2>
                  <p className="text-stone-500 font-sans mt-2 opacity-60">
                    諸大名との交渉を行い、同盟を締結して天下泰平の礎を築く。
                  </p>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 overflow-y-auto custom-scrollbar flex-1 pb-4">
                  {Object.values(FACTIONS)
                    .filter((f) => f !== FACTIONS.OTHER)
                    .map((factionName) => {
                      const relation =
                        diplomacyRelations[factionName] || "neutral";
                      const isAllied = relation === "alliance";
                      const isHostile = relation === "hostile";

                      return (
                        <div
                          key={factionName}
                          className={`bg-card-bg border ${
                            isAllied
                              ? "border-orange-500/50 shadow-[0_0_20px_rgba(249,115,22,0.1)]"
                              : isHostile
                                ? "border-red-500/50 shadow-[0_0_20px_rgba(239,68,68,0.1)]"
                                : "border-white/5"
                          } p-6 rounded-sm flex flex-col relative group hover:border-orange-500/30 transition-all`}
                        >
                          <div className="flex justify-between items-start mb-4">
                            <div
                              className={`px-2 py-1 rounded text-[9px] uppercase tracking-widest border ${
                                isAllied
                                  ? "bg-orange-500 text-black font-bold border-orange-500"
                                  : isHostile
                                    ? "bg-red-500/20 text-red-400 font-bold border-red-500/40 shadow-[0_0_10px_rgba(239,68,68,0.1)]"
                                    : "bg-white/5 text-stone-400 border-white/10"
                              }`}
                            >
                              {isAllied
                                ? "同盟中"
                                : isHostile
                                  ? "敵対"
                                  : "中立"}
                            </div>
                            <FactionKamon
                              faction={factionName as string}
                              className="w-8 h-8 opacity-30 group-hover:opacity-100 transition-opacity"
                            />
                          </div>

                          <h4 className="font-serif font-bold text-xl text-white mb-2">
                            {factionName}家
                          </h4>

                          <div className="mt-auto pt-6 space-y-3">
                            {!isAllied ? (
                              <button
                                onClick={() => {
                                  const cost = 5000;
                                  if (gold >= cost) {
                                    setGold((prev) => prev - cost);
                                    setDiplomacyRelations((prev) => ({
                                      ...prev,
                                      [factionName]: "alliance",
                                    }));
                                    addLog(
                                      `【同盟】${factionName}家と永久同盟を締結しました。`,
                                      "success",
                                    );
                                  } else {
                                    addLog(
                                      "【不足】同盟の契約金（5,000）が足りません。",
                                      "error",
                                    );
                                  }
                                }}
                                className="w-full bg-orange-500/10 border border-orange-500/30 py-3 text-[10px] font-bold tracking-widest text-orange-400 hover:bg-orange-500 hover:text-black transition-all flex items-center justify-center gap-2"
                              >
                                <Handshake className="w-3 h-3" /> 同盟締結
                                (5,000)
                              </button>
                            ) : (
                              <div className="space-y-2 w-full">
                                <button className="w-full bg-white/5 border border-white/10 py-3 text-[10px] font-bold tracking-widest text-stone-500 cursor-not-allowed flex items-center justify-center gap-2">
                                  同盟締結済み
                                </button>
                                <button
                                  onClick={() => {
                                    setDiplomacyRelations((prev) => {
                                      const next = { ...prev };
                                      delete next[factionName];
                                      return next;
                                    });
                                    addLog(
                                      `【同盟破棄】${factionName}家との同盟を破棄しました。`,
                                      "error",
                                    );

                                    // 同盟を破棄された大名家が所有する領地
                                    const enemyLands = lands.filter((l) => l.owner === factionName);
                                    if (enemyLands.length > 0) {
                                      const playerOwnedSet = new Set(occupiedLands);
                                      const adjacentPlayerLands: typeof lands = [];

                                      enemyLands.forEach((el) => {
                                        ROUTE_CONNECTIONS.forEach(([a, b]) => {
                                          let neighborId = -1;
                                          if (a === el.id) neighborId = b;
                                          else if (b === el.id) neighborId = a;

                                          if (neighborId !== -1 && playerOwnedSet.has(neighborId)) {
                                            const pl = lands.find((l) => l.id === neighborId);
                                            if (pl && !adjacentPlayerLands.some((existing) => existing.id === pl.id)) {
                                              adjacentPlayerLands.push(pl);
                                            }
                                          }
                                        });
                                      });

                                      if (adjacentPlayerLands.length > 0) {
                                        const targetLand = adjacentPlayerLands[Math.floor(Math.random() * adjacentPlayerLands.length)];
                                        const basePower = targetLand.power || 1000;
                                        // 激怒補正（兵力1.2倍〜1.7倍）
                                        const enemyPower = Math.floor(basePower * (1.2 + Math.random() * 0.5));

                                        setTimeout(() => {
                                          setInvasionEvent({
                                            land: targetLand,
                                            enemyFaction: factionName,
                                            enemyPower: Math.max(enemyPower, 500),
                                          });
                                          addLog(
                                            `【急告・遺恨襲撃】同盟破棄に激怒した${factionName}軍（総兵力 ${Math.max(enemyPower, 500).toLocaleString()}）が、隣接する我らが領国【${targetLand.name.split(" ")[0]}】へ即座に報復侵攻を仕掛けてきました！`,
                                            "error"
                                          );
                                        }, 1000);
                                      } else {
                                        // 直接隣接していないが、他の領内を通って/遠征して襲撃を仕掛ける
                                        const targetLand = lands.find((l) => occupiedLands.includes(l.id));
                                        if (targetLand) {
                                          const basePower = targetLand.power || 1000;
                                          const enemyPower = Math.floor(basePower * (1.0 + Math.random() * 0.4));
                                          setTimeout(() => {
                                            setInvasionEvent({
                                              land: targetLand,
                                              enemyFaction: factionName,
                                              enemyPower: Math.max(enemyPower, 500),
                                            });
                                            addLog(
                                              `【急告・遺恨襲撃】同盟を一方的に破棄された${factionName}家は我らに強い恨みを抱き、他領を通過・遠征して我らが国【${targetLand.name.split(" ")[0]}】へ報復軍（兵力 ${Math.max(enemyPower, 500).toLocaleString()}）を差し向けました！`,
                                              "error"
                                            );
                                          }, 1000);
                                        }
                                      }
                                    }
                                  }}
                                  className="w-full bg-red-950/20 border border-red-500/20 hover:bg-red-500 hover:text-black hover:border-red-500 transition-all py-2 text-[10px] font-bold tracking-widest text-red-400 flex items-center justify-center gap-2 rounded-sm"
                                >
                                  同盟を破棄する
                                </button>
                              </div>
                            )}
                          </div>
                        </div>
                      );
                    })}
                </div>
              </motion.div>
            )}

            {activeTab === "shop" && (
              <motion.div
                key="shop"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col p-8"
              >
                <div className="mb-8 flex flex-col md:flex-row md:items-center justify-between gap-4">
                  <div>
                    <h2 className="text-3xl font-serif font-bold text-gold tracking-widest uppercase flex items-center gap-4">
                      <Store className="w-8 h-8 opacity-50" /> 万屋
                    </h2>
                    <p className="text-stone-500 font-sans mt-2 opacity-60">
                      異国より持ち込まれた希少な兵装や、戦略の要となる宝物を取り扱う。まとめ買いも対応。
                    </p>
                  </div>
                  <div className="bg-white/5 border border-white/10 p-3 rounded flex items-center gap-6 text-[11px] font-bold">
                    <div className="flex items-center gap-2 text-gold">
                      <Crown className="w-4 h-4" />
                      <span>所持小判: {gold.toLocaleString()}枚</span>
                    </div>
                    <div className="flex items-center gap-2 text-amber-500">
                      <Coins className="w-4 h-4" />
                      <span>所持銅銭: {copper.toLocaleString()}文</span>
                    </div>
                  </div>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 overflow-y-auto custom-scrollbar flex-1 pb-4">
                  {MASTER_WEAPONS.map((wp) => {
                    const qty = shopQuantities[wp.id] || 1;
                    const goldPrice = wp.price * qty;
                    const copperPrice = (wp.price * 5) * qty;

                    return (
                      <div
                        key={wp.id}
                        className="bg-card-bg border border-white/5 p-6 rounded-sm relative group hover:border-gold/30 transition-all flex flex-col justify-between"
                      >
                        <div>
                          <div className="flex justify-between items-start mb-4">
                            <div className="bg-white/5 px-2 py-1 rounded text-[9px] text-stone-400 uppercase tracking-widest border border-white/10">
                              {wp.type}
                            </div>
                            <RarityStars count={wp.rarity} />
                          </div>
                          <h4 className="font-serif font-bold text-xl text-white mb-2 group-hover:text-gold transition-colors">
                            {wp.name}
                          </h4>
                          {wp.description ? (
                            <p className="text-[10px] text-stone-500 mb-6 italic leading-relaxed min-h-[30px]">
                              {wp.description}
                            </p>
                          ) : (
                            <div className="flex gap-4 mb-6 text-[10px] uppercase tracking-tighter text-stone-500 min-h-[30px]">
                              <span>武力: +{wp.atk}</span>
                              <span>知略: +{wp.int}</span>
                              <span>統率: +{wp.def}</span>
                            </div>
                          )}
                        </div>

                        <div className="mt-4 border-t border-white/5 pt-4">
                          {/* Stepper block */}
                          <div className="mb-4">
                            <div className="flex items-center justify-between text-[10px] text-stone-400 font-bold mb-2">
                              <span>購入数量（まとめ買い）</span>
                              <span className="text-gold font-mono text-xs">{qty} 個</span>
                            </div>
                            <div className="flex items-center gap-1.5 flex-wrap">
                              <button
                                onClick={() =>
                                  setShopQuantities((prev) => ({
                                    ...prev,
                                    [wp.id]: Math.max(1, qty - 1),
                                  }))
                                }
                                className="bg-white/5 hover:bg-white/15 px-2 py-1 border border-white/10 text-stone-300 text-[10px] rounded"
                              >
                                -1
                              </button>
                              <button
                                onClick={() =>
                                  setShopQuantities((prev) => ({
                                    ...prev,
                                    [wp.id]: Math.max(1, qty - 5),
                                  }))
                                }
                                className="bg-white/5 hover:bg-white/15 px-2 py-1 border border-white/10 text-stone-300 text-[10px] rounded"
                              >
                                -5
                              </button>
                              <button
                                onClick={() =>
                                  setShopQuantities((prev) => ({
                                    ...prev,
                                    [wp.id]: Math.min(99, qty + 1),
                                  }))
                                }
                                className="bg-white/5 hover:bg-white/15 px-2 py-1 border border-white/10 text-stone-300 text-[10px] rounded"
                              >
                                +1
                              </button>
                              <button
                                onClick={() =>
                                  setShopQuantities((prev) => ({
                                    ...prev,
                                    [wp.id]: Math.min(99, qty + 5),
                                  }))
                                }
                                className="bg-white/5 hover:bg-white/15 px-2 py-1 border border-white/10 text-stone-300 text-[10px] rounded"
                              >
                                +5
                              </button>
                              <button
                                onClick={() =>
                                  setShopQuantities((prev) => ({
                                    ...prev,
                                    [wp.id]: Math.min(99, qty + 10),
                                  }))
                                }
                                className="bg-white/5 hover:bg-white/15 px-2 py-1 border border-white/10 text-stone-300 text-[10px] rounded"
                              >
                                +10
                              </button>
                              <span className="text-stone-600 px-1">|</span>
                              <button
                                onClick={() =>
                                  setShopQuantities((prev) => ({
                                    ...prev,
                                    [wp.id]: 1,
                                  }))
                                }
                                className="bg-[#1C1A16] hover:bg-stone-800 text-[9px] text-amber-500 font-bold px-1.5 py-1 border border-amber-500/20 rounded"
                              >
                                戻す
                              </button>
                            </div>
                          </div>

                          {/* Purchase Buttons */}
                          <div className="grid grid-cols-2 gap-2">
                            <button
                              onClick={() => handleBuyItem(wp, qty, "gold")}
                              disabled={gold < goldPrice}
                              className={`py-2 rounded-sm text-[10px] font-bold tracking-wider transition-all flex flex-col items-center justify-center border ${
                                gold >= goldPrice
                                  ? "bg-amber-950/20 border-gold/40 hover:bg-gold hover:text-black text-gold font-bold"
                                  : "bg-stone-900 border-white/5 text-stone-600 cursor-not-allowed"
                              }`}
                            >
                              <span className="text-[8px] opacity-75">小判で決済</span>
                              <span className="flex items-center gap-1 font-mono text-[11px] mt-0.5">
                                <Crown className="w-3 h-3 shrink-0" />
                                {goldPrice.toLocaleString()}枚
                              </span>
                            </button>

                            <button
                              onClick={() => handleBuyItem(wp, qty, "copper")}
                              disabled={copper < copperPrice}
                              className={`py-2 rounded-sm text-[10px] font-bold tracking-wider transition-all flex flex-col items-center justify-center border ${
                                copper >= copperPrice
                                  ? "bg-emerald-950/20 border-emerald-500/40 hover:bg-emerald-500 hover:text-black text-emerald-300 font-bold animate-pulse"
                                  : "bg-stone-900 border-white/5 text-stone-600 cursor-not-allowed"
                              }`}
                            >
                              <span className="text-[8px] opacity-75">銅銭で決済</span>
                              <span className="flex items-center gap-1 font-mono text-[11px] mt-0.5">
                                <Coins className="w-3 h-3 shrink-0" />
                                {copperPrice.toLocaleString()}文
                              </span>
                            </button>
                          </div>
                        </div>
                      </div>
                    );
                  })}
                </div>
              </motion.div>
            )}

            {activeTab === "item" && (
              <motion.div
                key="item"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col p-8"
              >
                <div className="mb-8 flex flex-col md:flex-row md:items-center justify-between gap-4">
                  <div>
                    <h2 className="text-3xl font-serif font-bold text-gold tracking-widest uppercase flex items-center gap-4">
                      <Package className="w-8 h-8 opacity-50" /> 兵装蔵
                    </h2>
                    <p className="text-stone-500 font-sans mt-2 opacity-60">
                      所持している軍略資産および兵装（武器）の在庫を管理・使用する。
                    </p>
                  </div>
                </div>

                <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-6 overflow-y-auto custom-scrollbar flex-1 pb-4">
                  {(() => {
                    // Gather tools (consumable) from items state
                    const tools = items.filter((itm) => itm && itm.type === "道具");
                    const groupedTools: Record<string, any> = {};
                    tools.forEach((itm) => {
                      if (!groupedTools[itm.id]) {
                        groupedTools[itm.id] = { ...itm, count: 0, list: [] };
                      }
                      groupedTools[itm.id].count += 1;
                      groupedTools[itm.id].list.push(itm);
                    });

                    // Gather weapon stock from inventoryWeapons
                    const weaponItems = Object.entries(inventoryWeapons)
                      .filter(([id, count]) => (count as number) > 0)
                      .map(([id, count]) => {
                        const baseWp = MASTER_WEAPONS.find((w) => w.id === id);
                        return baseWp ? { ...baseWp, count: count as number, isWeapon: true } : null;
                      })
                      .filter((w): w is any => w !== null);

                    const unifiedList = [
                      ...Object.values(groupedTools),
                      ...weaponItems,
                    ];

                    if (unifiedList.length === 0) {
                      return (
                        <div className="col-span-full h-64 flex flex-col items-center justify-center text-stone-800 opacity-20">
                          <Package className="w-16 h-16 mb-4" />
                          <span className="font-serif tracking-widest">
                            所持品なし
                          </span>
                        </div>
                      );
                    }

                    return unifiedList.map((item, idx) => (
                      <div
                        key={idx}
                        className="bg-card-bg border border-white/5 p-5 rounded-sm flex flex-col justify-between hover:border-gold/20 transition-all font-sans relative group"
                      >
                        {/* Count Badge */}
                        <div className="absolute top-3 right-3 bg-gold/15 border border-gold/30 text-gold font-mono text-[10px] px-2 py-0.5 rounded-sm font-bold">
                          所持: {item.count}
                        </div>

                        <div>
                          <div className="flex gap-2 items-center mb-3">
                            <span className="bg-white/5 px-1.5 py-0.5 rounded text-[8px] text-stone-400 border border-white/10 uppercase tracking-wider">
                              {item.type}
                            </span>
                            <RarityStars count={item.rarity} />
                          </div>

                          <h4 className="font-serif font-bold text-base text-white mb-2 truncate pr-16">
                            {item.name}
                          </h4>

                          {item.description ? (
                            <p className="text-[10px] text-stone-400 mb-4 italic leading-relaxed min-h-[40px]">
                              {item.description}
                            </p>
                          ) : (
                            <div className="flex gap-3 mb-4 text-[10px] font-bold text-stone-400 border-t border-b border-white/5 py-1.5 min-h-[40px]">
                              {item.atk !== 0 && <span>武力 +{item.atk}</span>}
                              {item.int !== 0 && <span>知略 +{item.int}</span>}
                              {item.def !== 0 && <span>統率 +{item.def}</span>}
                            </div>
                          )}
                        </div>

                        <div className="mt-4 border-t border-white/5 pt-4">
                          {item.isWeapon ? (
                            <div className="text-center py-2 text-[9px] text-stone-600 font-serif leading-normal bg-black/20 rounded">
                              ※「部隊」の各武将の「養成・兵装」から装備させることができます。
                            </div>
                          ) : (
                            <div className="space-y-2">
                              {/* Single Use */}
                              <button
                                onClick={() => handleUseItem(item.list[0])}
                                className="w-full bg-gold/15 border border-gold/30 text-gold text-[10px] py-2 rounded-sm hover:bg-gold hover:text-black transition-all font-bold tracking-widest"
                              >
                                1個使用する
                              </button>

                              {/* Bulk Use */}
                              {item.count > 1 && (
                                <button
                                  onClick={() => handleUseItemBulkByItemType(item.id, item.count)}
                                  className="w-full bg-[#1C1F1E] border border-emerald-500/30 text-emerald-400 text-[10px] py-1.5 rounded-sm hover:bg-emerald-500 hover:text-black transition-all font-bold tracking-widest flex items-center justify-center gap-1.5"
                                >
                                  <span>⚡</span> {item.count}個 まとめて一括使用
                                </button>
                              )}
                            </div>
                          )}
                        </div>
                      </div>
                    ));
                  })()}
                </div>
              </motion.div>
            )}

            {activeTab === "advisor" && (
              <motion.div
                key="advisor"
                className="h-full flex flex-col bg-card-bg rounded-sm border border-white/5 p-12"
              >
                <h2 className="text-4xl font-serif font-bold text-gold mb-8 flex items-center gap-4 tracking-widest uppercase">
                  <Sparkles className="w-8 h-8 opacity-50" /> 軍師
                </h2>
                <div className="flex-1 bg-dark-bg border border-white/5 p-8 mb-8 overflow-y-auto custom-scrollbar flex flex-col gap-6 relative shadow-inner">
                  <div className="absolute top-4 right-6 opacity-5 uppercase font-serif text-6xl tracking-tighter">
                    軍師の献策
                  </div>
                  {advisorResponse ? (
                    <div className="flex gap-6 animate-fade-in z-10">
                      <div className="w-14 h-14 bg-white/5 rounded-sm flex items-center justify-center border border-white/10 shrink-0 shadow-lg">
                        <Users className="w-6 h-6 text-gold" />
                      </div>
                      <div className="bg-transparent p-6 text-stone-300 leading-relaxed font-serif italic text-lg border-l-2 border-gold/30">
                        {advisorResponse}
                      </div>
                    </div>
                  ) : (
                    <div className="h-full flex items-center justify-center text-stone-700 text-lg font-serif italic opacity-30">
                      "御下命を、御屋形様。"
                    </div>
                  )}
                </div>
                <div className="flex gap-6">
                  <input
                    type="text"
                    value={advisorQuery}
                    onChange={(e) => setAdvisorQuery(e.target.value)}
                    onKeyDown={(e) =>
                      e.key === "Enter" &&
                      !isConsulting &&
                      handleConsultAdvisor()
                    }
                    placeholder="軍師に問う（例：天下布武への計略を）"
                    className="flex-1 bg-dark-bg border border-white/10 px-8 py-5 text-white outline-none focus:border-gold transition-colors font-sans font-light tracking-wide placeholder:text-stone-700"
                  />
                  <button
                    onClick={handleConsultAdvisor}
                    disabled={isConsulting || !advisorQuery.trim()}
                    className="bg-gold hover:bg-[#D4A159] px-12 py-5 font-bold text-black transition-all disabled:opacity-30 uppercase tracking-[0.2em] text-xs shadow-[0_0_30px_rgba(197,160,89,0.2)]"
                  >
                    {isConsulting ? "思案中..." : "相談"}
                  </button>
                </div>
              </motion.div>
            )}
            {activeTab === "navy" && (
              <motion.div
                key="navy"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col bg-card-bg rounded-sm border border-white/5 p-12"
              >
                <div className="flex flex-col lg:flex-row justify-between items-start gap-4 mb-8">
                  <div>
                    <h2 className="text-4xl font-serif font-bold text-cyan-400 flex items-center gap-4 tracking-widest uppercase">
                      <Anchor className="w-8 h-8 text-cyan-400 animate-pulse" />{" "}
                      水軍・海港管理
                    </h2>
                    <p className="text-stone-500 font-sans mt-2 text-sm leading-relaxed max-w-3xl">
                      制海権を掌握し、強力な戦国水軍を編制せよ。開港と港湾等級の拡張は莫大な特別関税と物流収入をもたらし、合戦時には沿岸より焙烙火矢と大筒による援護射撃の加護（兵力ボーナス）を受けられます。
                    </p>
                  </div>

                  {/* 海軍新設ボタン */}
                  {!hasNavy && (
                    <button
                      onClick={() => {
                        if (copper >= 5000 && gold >= 1000) {
                          setCopper((prev) => prev - 5000);
                          setGold((prev) => prev - 1000);
                          setHasNavy(true);
                          addLog(
                            "【水軍旗起立】祝、直属日本丸水軍の創立！天下統一に向けて沿岸の支配および開港・海上交易の道を拓きました！",
                            "success",
                          );
                        } else {
                          addLog(
                            "【創立不可】水軍新設に必要な国費（金1,000 / 銅5,000）が不足しております。",
                            "error",
                          );
                        }
                      }}
                      className="bg-cyan-500 text-black hover:bg-cyan-400 px-10 py-4 font-serif font-bold text-sm tracking-[0.2em] transition-all shadow-[0_0_20px_rgba(34,211,238,0.4)] animate-pulse rounded-sm shrink-0 uppercase select-none"
                    >
                      日本丸水軍を創設する (費用: 金1,000 / 銅5,000)
                    </button>
                  )}
                </div>

                {!hasNavy ? (
                  <div className="flex-1 border border-white/5 bg-black/40 rounded-sm p-12 flex flex-col items-center justify-center text-center">
                    <Ship className="w-20 h-20 text-stone-700 mb-6 opacity-20 animate-bounce" />
                    <h3 className="text-lg font-serif font-bold text-stone-500 mb-2">
                      水軍が未創設です
                    </h3>
                    <p className="text-xs text-stone-600 max-w-md leading-relaxed font-sans">
                      直属の水軍を新設・組織することで、造船所での「千石船」「関船」「安宅船」等の軍船起工・就役、および占領した海港の「港湾造成による格式向上（交易益発生）」が許可されます。
                    </p>
                  </div>
                ) : (
                  <div className="flex-1 flex flex-col space-y-8 overflow-y-auto custom-scrollbar pr-2 p-1">
                    {/* 上部ステータスカード */}
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                      <div className="bg-dark-bg/80 border border-cyan-500/20 p-5 rounded-sm flex flex-col justify-between shadow-lg relative overflow-hidden">
                        <div className="absolute top-0 right-0 p-8 opacity-5 text-cyan-400 font-serif text-5xl font-bold">
                          海
                        </div>
                        <div className="text-stone-400 text-[10px] font-mono tracking-widest block uppercase text-cyan-400/80 font-bold mb-2 font-bold mb-2">
                          ● 水軍防衛線
                        </div>
                        <div className="text-2xl font-serif text-white font-bold mb-1">
                          制海権掌握済
                        </div>
                        <p className="text-[11px] text-stone-500 font-light font-light">
                          全沿岸を我が航路で警備中。海岸沿いの領地戦で水軍の援護砲撃が発生します。
                        </p>
                      </div>

                      <div className="bg-dark-bg/80 border border-cyan-500/20 p-5 rounded-sm flex flex-col justify-between shadow-lg relative overflow-hidden">
                        <div className="absolute top-0 right-0 p-8 opacity-5 text-cyan-400 font-serif text-5xl font-bold font-bold">
                          撃
                        </div>
                        <div className="text-stone-400 text-[10px] font-mono tracking-widest block uppercase text-cyan-400/80 font-bold mb-2">
                          ● 砲撃加勢戦力
                        </div>
                        <div className="text-2xl font-mono text-cyan-300 font-bold mb-1">
                          +
                          {(
                            1000 +
                            (navyShips?.kobaya || 0) * 100 +
                            (navyShips?.sekibune || 0) * 350 +
                            (navyShips?.atakebune || 0) * 1200
                          ).toLocaleString()}
                        </div>
                        <p className="text-[11px] text-stone-500 font-light">
                          焙烙大筒支援の基礎戦力。(小早川 +100 / 関船 +350 /
                          安宅船 +1,200)
                        </p>
                      </div>

                      <div className="bg-dark-bg/80 border border-cyan-500/20 p-5 rounded-sm flex flex-col justify-between shadow-lg relative overflow-hidden">
                        <div className="absolute top-0 right-0 p-8 opacity-5 text-cyan-400 font-serif text-5xl font-bold">
                          税
                        </div>
                        <div className="text-stone-400 text-[10px] font-mono tracking-widest block uppercase text-cyan-400/80 font-bold mb-2">
                          ● 物流貿易ブースト
                        </div>
                        <div className="text-2xl font-mono text-emerald-400 font-bold mb-1">
                          +{((navyShips?.sengokubune || 0) * 10).toFixed(0)}%
                        </div>
                        <p className="text-[11px] text-stone-500 font-light font-light">
                          千石船を稼働させることによる物流国益効果。徴税時の年貢回収率を倍率加算。
                        </p>
                      </div>
                    </div>

                    {/* 海路統一・海上覇権海図 ＆ 港湾戦略閣 */}
                    <div className="grid grid-cols-1 xl:grid-cols-12 gap-8 flex-1">
                      
                      {/* Column 1: Nautical Sea Chart Map (7/12 width) */}
                      <div className="xl:col-span-7 bg-[#06121a] border border-cyan-500/30 rounded p-4 flex flex-col relative overflow-hidden group select-none shadow-[inside_0_0_50px_rgba(6,182,212,0.1)] min-h-[460px]">
                        
                        {/* Map Header details */}
                        <div className="z-10 flex justify-between items-center bg-black/60 px-4 py-2 rounded border border-white/5 backdrop-blur-sm">
                          <div className="flex items-center gap-2">
                            <MapIcon className="w-3.5 h-3.5 text-cyan-400 animate-pulse" />
                            <span className="font-serif font-bold text-[11px] text-cyan-300 tracking-wider">
                              日本近海海事航路配備図 (Sengoku Nautical Chart)
                            </span>
                          </div>
                          <span className="text-[9px] font-mono text-cyan-500/80">SCALE: 1:4,200,000</span>
                        </div>

                        {/* Compass Rose */}
                        <div className="absolute right-6 bottom-6 opacity-10 pointer-events-none text-cyan-400 flex flex-col items-center">
                          <div className="w-20 h-20 rounded-full border-2 border-dashed border-cyan-400/80 flex items-center justify-center animate-spin-slow">
                            <Anchor className="w-8 h-8 text-cyan-400 rotate-45" />
                          </div>
                          <span className="text-[10px] font-serif mt-1 font-bold tracking-widest text-cyan-400/80">外海諸島</span>
                        </div>

                        {/* Sea wave ripples */}
                        <div className="absolute inset-0 bg-ocean opacity-30 pointer-events-none"></div>

                        {/* Chart Body Canvas Stage */}
                        <div className="relative flex-1 w-full h-full mt-4 rounded border border-white/5 overflow-hidden bg-black/30 min-h-[300px]">
                          {/* Maritime Patrol Route SVG Lines */}
                          <svg className="absolute inset-0 w-full h-full pointer-events-none">
                            {(() => {
                              const connectionsList = [
                                [73, 1], // 琉球 <-> 薩摩
                                [1, 4],  // 薩摩 <-> 土佐
                                [2, 72], // 肥前 <-> 対馬
                                [2, 67], // 肥前 <-> 筑前
                                [67, 22],// 筑前 <-> 豊後
                                [22, 28],// 豊後 <-> 伊予
                                [25, 47],// 長門 <-> 石見
                                [25, 26],// 長門 <-> 備前
                                [47, 5], // 石見 <-> 播磨
                                [26, 31],// 備前 <-> 摂津
                                [28, 31],// 伊予 <-> 摂津
                                [31, 34],// 摂津 <-> 駿河
                                [34, 8], // 駿河 <-> 相模
                                [8, 41], // 相模 <-> 常陸
                                [41, 46],// 常陸 <-> 陸前
                                [13, 46],// 越後 <-> 陸前
                                [46, 19] // 陸前 <-> 蝦夷
                              ];

                              return connectionsList.map(([id1, id2], idx) => {
                                const p1 = lands.find((l) => l.id === id1);
                                const p2 = lands.find((l) => l.id === id2);
                                if (!p1 || !p2) return null;

                                const isSelectedRoute =
                                  selectedNavyPortId === id1 || selectedNavyPortId === id2;

                                return (
                                  <line
                                    key={idx}
                                    x1={`${p1.x}%`}
                                    y1={`${p1.y}%`}
                                    x2={`${p2.x}%`}
                                    y2={`${p2.y}%`}
                                    className={`transition-all ${
                                      isSelectedRoute
                                        ? "stroke-cyan-400 stroke-2"
                                        : "stroke-cyan-500/20 stroke-[1.5]"
                                    }`}
                                    style={{
                                      strokeDasharray: isSelectedRoute ? "6,4" : "4,4",
                                      filter: isSelectedRoute ? "drop-shadow(0px 0px 4px rgba(34,211,238,0.6))" : "none"
                                    }}
                                  />
                                );
                              });
                            })()}
                          </svg>

                          {/* Port coordinate nodes */}
                          {lands
                            .filter((l) => l.isPort)
                            .map((land) => {
                              const isOwned = occupiedLands.includes(land.id);
                              const isAlliance =
                                diplomacyRelations[land.owner] === "alliance" && !isOwned;
                              const isSelected = selectedNavyPortId === land.id;

                              let ringColor = "border-red-600 shadow-[0_0_10px_rgba(220,38,38,0.5)] bg-red-950/90 hover:bg-red-900";

                              if (isOwned) {
                                ringColor = "border-cyan-400 shadow-[0_0_12px_rgba(34,211,238,0.7)] bg-cyan-950/90 hover:bg-cyan-900 animate-pulse";
                              } else if (isAlliance) {
                                ringColor = "border-emerald-500 shadow-[0_0_10px_rgba(16,185,129,0.5)] bg-emerald-950/90 hover:bg-emerald-900";
                              }

                              return (
                                <button
                                  key={land.id}
                                  onClick={() => setSelectedNavyPortId(land.id)}
                                  className="absolute -translate-x-1/2 -translate-y-1/2 cursor-pointer z-20 group/node"
                                  style={{ left: `${land.x}%`, top: `${land.y}%` }}
                                >
                                  {/* Pulsing ring for selection */}
                                  {isSelected && (
                                    <div className="absolute inset-0 -m-3 rounded-full border-2 border-cyan-400 animate-ping opacity-70 pointer-events-none"></div>
                                  )}

                                  <div
                                    className={`w-4 h-4 rounded-full border flex items-center justify-center transition-all duration-300 relative ${
                                      isSelected
                                        ? "scale-140 border-gold shadow-[0_0_15px_#C5A059] bg-stone-900"
                                        : ringColor
                                    }`}
                                  >
                                    <Anchor className={`w-2 h-2 ${isSelected ? "text-gold" : "text-white"}`} />
                                  </div>

                                  {/* Mini floating label */}
                                  <div className={`absolute left-5 top-1/2 -translate-y-1/2 bg-black/80 px-1.5 py-0.5 rounded text-[8px] font-serif font-bold whitespace-nowrap opacity-60 group-hover/node:opacity-100 transition-opacity border pointer-events-none z-30 ${
                                    isSelected ? "border-gold text-gold opacity-100" : "border-white/5 text-stone-300"
                                  }`}>
                                    {land.name.split(" ")[0]}
                                  </div>
                                </button>
                              );
                            })}
                        </div>

                        {/* Legend */}
                        <div className="mt-4 flex gap-4 text-[9px] font-serif justify-center text-stone-400 bg-stone-950/40 py-2 rounded border border-white/5">
                          <span className="flex items-center gap-1">
                            <span className="w-2.5 h-2.5 rounded-full bg-cyan-950 border border-cyan-400 shadow-[0_0_6px_rgba(34,211,238,0.5)]"></span>
                            支配海港
                          </span>
                          <span className="flex items-center gap-1">
                            <span className="w-2.5 h-2.5 rounded-full bg-emerald-950 border border-emerald-500 shadow-[0_0_6px_rgba(16,185,129,0.5)]"></span>
                            盟友交易港
                          </span>
                          <span className="flex items-center gap-1">
                            <span className="w-2.5 h-2.5 rounded-full bg-red-950 border border-red-600 shadow-[0_0_6px_rgba(220,38,38,0.5)]"></span>
                            敵対占領港
                          </span>
                          <span className="flex items-center gap-1 text-gold font-bold">
                            <span>★</span>
                            選択中
                          </span>
                        </div>
                      </div>

                      {/* Column 2: Operations Command Desk (5/12 width) */}
                      <div className="xl:col-span-5 bg-card-bg border border-white/5 p-6 rounded flex flex-col justify-between min-h-[460px]">
                        
                        {(() => {
                          const sLand = lands.find((l) => l.id === selectedNavyPortId) || lands.find((l) => l.isPort);
                          if (!sLand) {
                            return (
                              <div className="flex-1 flex flex-col items-center justify-center text-center text-stone-600">
                                <Anchor className="w-12 h-12 text-stone-700 mb-2 opacity-50" />
                                <p className="text-xs font-serif">海図から目的地を指示してくだされ</p>
                              </div>
                            );
                          }

                          const isOwned = occupiedLands.includes(sLand.id);
                          const isAlliance =
                            diplomacyRelations[sLand.owner] === "alliance" && !isOwned;
                          const harborLvl = sLand.harborLevel || 0;
                          const isMaxLvl = harborLvl >= 5;
                          const buildCostGold = 1000 + harborLvl * 500;
                          const buildCostCopper = 5000 + harborLvl * 2500;
                          const canUpgrade =
                            gold >= buildCostGold &&
                            copper >= buildCostCopper &&
                            !isMaxLvl;

                          return (
                            <div className="flex-1 flex flex-col justify-between h-full">
                              <div>
                                <div className="flex items-center justify-between border-b border-white/5 pb-3 mb-4">
                                  <div className="text-left font-serif">
                                    <span className="text-[10px] uppercase text-cyan-400 font-bold block mb-0.5 tracking-widest">
                                      ⚓ PORT DESK
                                    </span>
                                    <h3 className="text-2xl font-bold text-white flex items-center gap-2">
                                      {sLand.name.split(" ")[0]}
                                      <span className="text-xs font-sans font-normal text-stone-500">
                                        {sLand.name.split(" ")[1] || ""}
                                      </span>
                                    </h3>
                                  </div>
                                  <span className={`px-2.5 py-1 text-xs font-serif font-bold rounded-sm border ${
                                    isOwned
                                      ? "bg-cyan-950/40 text-cyan-300 border-cyan-800"
                                      : isAlliance
                                        ? "bg-emerald-950/40 text-emerald-300 border-emerald-800"
                                        : "bg-red-950/40 text-red-400 border-red-800"
                                  }`}>
                                    {isOwned ? "我が領地" : isAlliance ? "同盟領国" : `${sLand.owner}領`}
                                  </span>
                                </div>

                                <div className="space-y-4 font-sans text-left">
                                  {/* Stats details table */}
                                  <div className="bg-black/30 border border-white/5 p-4 rounded text-xs space-y-2 font-sans">
                                    <div className="flex justify-between font-sans">
                                      <span className="text-stone-500 font-sans">防守城郭格式:</span>
                                      <span className="font-serif font-bold text-stone-200">★ {sLand.level}</span>
                                    </div>
                                    <div className="flex justify-between font-sans">
                                      <span className="text-stone-500 font-sans">守備兵力抵抗:</span>
                                      <span className="text-red-400 font-mono font-bold">{sLand.power.toLocaleString()} 兵</span>
                                    </div>
                                    <div className="flex justify-between border-t border-white/5 pt-2 font-sans">
                                      <span className="text-stone-500 font-sans">水軍格式等級:</span>
                                      <span className="text-cyan-400 font-bold">
                                        {harborLvl === 0 ? "未造成港港口" : `海港格式 等級${harborLvl}`}
                                      </span>
                                    </div>
                                    <div className="flex justify-between font-sans">
                                      <span className="text-stone-500 font-sans">毎徴税時交易果:</span>
                                      <span className="text-emerald-400 font-mono font-bold">
                                        {harborLvl === 0
                                          ? "未配備"
                                          : `小判 +${(harborLvl * 100).toLocaleString()}枚 / 銅銭 +${(harborLvl * 1200).toLocaleString()}貫`}
                                      </span>
                                    </div>
                                  </div>

                                  {/* Action logic decision UI blocks */}
                                  {isOwned ? (
                                    /* CASE A: Port upgrades */
                                    <div className="pt-2">
                                      <h4 className="text-[11px] font-bold text-gold uppercase tracking-wider mb-2 font-serif">
                                        港湾拡張造成指示 (Upgrade Coastal Infrastructure)
                                      </h4>
                                      <p className="text-[10px] text-stone-400 leading-relaxed mb-4">
                                        現在の港湾格式等級を引き上げ、物流・関税収入を底上げします。全港を造成することで、天下統一時の兵站物流の強靭な礎となります。
                                      </p>
                                      
                                      {!isMaxLvl ? (
                                        <div className="bg-black/40 border border-stone-800 p-3 rounded mb-4 text-[10px] space-y-1 text-stone-400 font-sans">
                                          <div className="flex justify-between font-sans">
                                            <span>造成費用 (小判):</span>
                                            <span className={gold >= buildCostGold ? "text-amber-400 font-semibold font-sans" : "text-red-500 font-sans"}>
                                              -{buildCostGold.toLocaleString()}枚 (保有: {gold.toLocaleString()})
                                            </span>
                                          </div>
                                          <div className="flex justify-between font-sans">
                                            <span>造成費用 (銅銭):</span>
                                            <span className={copper >= buildCostCopper ? "text-amber-400 font-semibold font-sans" : "text-red-500 font-sans"}>
                                              -{buildCostCopper.toLocaleString()}貫 (保有: {copper.toLocaleString()})
                                            </span>
                                          </div>
                                        </div>
                                      ) : (
                                        <div className="bg-[#1c1917]/60 border border-cyan-800 p-3 rounded text-[10px] text-cyan-400 text-center font-serif leading-relaxed mb-4">
                                          この港湾は最上格式格「天下屈指の軍港交易港」に造成完了いたしました！
                                        </div>
                                      )}

                                      <button
                                        disabled={isMaxLvl || !canUpgrade}
                                        onClick={() => {
                                          if (canUpgrade) {
                                            setGold((prev) => prev - buildCostGold);
                                            setCopper((prev) => prev - buildCostCopper);

                                            const updatedLands = lands.map((l) => {
                                              if (l.id === sLand.id) {
                                                return {
                                                  ...l,
                                                  harborLevel: (l.harborLevel || 0) + 1,
                                                };
                                              }
                                              return l;
                                            });
                                            setLands(updatedLands);
                                            addLog(
                                              `【港湾造成】${sLand.name.split(" ")[0]} の造成を拡大し、海港格式等級を ${harborLvl + 1} に高めました！`,
                                              "success",
                                            );
                                          } else if (!isMaxLvl) {
                                            addLog(
                                              "【造成失敗】港湾造成に必要な国庫の資金が不足しております。",
                                              "error",
                                            );
                                          }
                                        }}
                                        className={`w-full py-3 text-xs font-serif font-bold rounded-sm border select-none tracking-widest cursor-pointer transition-all ${
                                          isMaxLvl
                                            ? "bg-amber-950/20 border-amber-900/20 text-amber-500 cursor-default"
                                            : canUpgrade
                                              ? "bg-cyan-950 border-cyan-500 text-cyan-400 hover:bg-cyan-400 hover:text-black hover:border-cyan-400"
                                              : "bg-[#292524] border-white/5 text-stone-500 cursor-not-allowed"
                                        }`}
                                      >
                                        {isMaxLvl ? "最高造成格式" : harborLvl === 0 ? "開港造成開始" : "港湾格式を拡張する"}
                                      </button>
                                    </div>
                                  ) : isAlliance ? (
                                    /* CASE B: Allied peace trade */
                                    <div className="pt-2 bg-stone-950/40 p-4 rounded border border-emerald-900/10">
                                      <h4 className="text-[11px] font-bold text-emerald-400 uppercase tracking-wider mb-2 font-serif flex items-center gap-1">
                                        🤝 同盟安全航路領
                                      </h4>
                                      <p className="text-[10px] text-stone-400 leading-relaxed">
                                        {sLand.owner}家は我が同盟国であるため、海上哨戒および渡海進撃は一切不要です。この港口は同盟国との「安全交易窓口」として機能し、海上封鎖を解除しております。
                                      </p>
                                    </div>
                                  ) : (
                                    /* CASE C: Hostile assault */
                                    <div className="pt-2 font-sans">
                                      <h4 className="text-[11px] font-bold text-red-400 uppercase tracking-wider mb-2 font-serif flex items-center gap-1">
                                        🔱 海上遠征水上強襲指示 (Launch Maritime Expedition)
                                      </h4>
                                      <p className="text-[10px] text-stone-400 leading-relaxed mb-4">
                                        陸路による他郭境界を一切跳び越し、我が大編隊を渡海遠征させます。制海権を掌握するために港湾都市を奇襲侵略せよ！
                                      </p>

                                      <div className="flex items-center justify-between text-[11px] mb-4 font-sans">
                                        <span className="text-stone-500 font-sans">攻撃派遣大隊:</span>
                                        <select
                                          id={`navy-assault-select-${sLand.id}`}
                                          className="bg-[#121212] border border-stone-800 rounded text-stone-300 text-[10px] px-2 py-1 outline-none focus:border-cyan-500 cursor-pointer font-sans"
                                        >
                                          <option value={0}>第1特設部隊 (戦力: {calculateDeckPower(0).toLocaleString()})</option>
                                          <option value={1}>第2特設部隊 (戦力: {calculateDeckPower(1).toLocaleString()})</option>
                                          <option value={2}>第3特設部隊 (戦力: {calculateDeckPower(2).toLocaleString()})</option>
                                        </select>
                                      </div>

                                      <button
                                        onClick={() => {
                                          const select = document.getElementById(
                                            `navy-assault-select-${sLand.id}`,
                                          ) as HTMLSelectElement;
                                          const idx = select ? Number(select.value) : 0;
                                          addLog(
                                            `【海上遠征】我が海軍の海上哨戒大艦隊を編成！港湾【${sLand.name.split(" ")[0]}】に向けて出帆いたしました！`,
                                            "success",
                                            );
                                          startBattle(sLand, idx);
                                        }}
                                        className="w-full py-3 bg-gradient-to-r from-red-950 to-red-900 hover:from-cyan-950 hover:to-cyan-900 border border-red-800 hover:border-cyan-500 text-red-300 hover:text-cyan-400 rounded transition-all font-serif font-bold text-xs tracking-widest text-center shadow-[0_4px_12px_rgba(239,68,68,0.15)] cursor-pointer"
                                      >
                                        水軍を派遣し強襲せよ！
                                      </button>
                                    </div>
                                  )}
                                </div>
                              </div>
                            </div>
                          );
                        })()}
                      </div>
                    </div>

                    {/* 造船所と港湾造成 */}
                    <div className="grid grid-cols-1 xl:grid-cols-2 gap-8 flex-1">
                      {/* 造船所 */}
                      <div className="bg-[#1c1917]/40 border border-white/5 p-6 rounded-sm flex flex-col">
                        <h3 className="font-serif font-bold text-lg text-cyan-300 mb-2 pb-2 border-b border-white/5 flex items-center gap-2">
                          <Ship className="w-5 h-5 text-cyan-400" />{" "}
                          軍船起工造船所 (Build Warships)
                        </h3>
                        <p className="text-xs text-stone-500 mb-6 leading-relaxed font-sans">
                          木材・銑鉄・銅銭を投下して大小軍船を新造・稼働。造船された軍船は、水軍火力や海運輸送力を永久的に累計向上させます。
                        </p>

                        <div className="space-y-4 flex-1 overflow-y-auto custom-scrollbar max-h-[460px] pr-2">
                          {[
                            {
                              id: "sengokubune",
                              name: "千石船 (Sengokubune)",
                              desc: "大型の商用物流和船。巨万の交易荷物を安全に回航させ、年貢徴収時の全収入を【1隻につき +10%】恒常的に向上させます。",
                              costGold: 400,
                              costCopper: 2500,
                              count: navyShips?.sengokubune || 0,
                              icon: "🏮",
                            },
                            {
                              id: "kobaya",
                              name: "小早川 (Kobaya)",
                              desc: "軽量俊敏な小型軍船。素早い回遊による航路哨戒を担い、合戦時に敵を攪乱【兵力加算: +100】の焙烙投げ支援を行います。",
                              costGold: 100,
                              costCopper: 1000,
                              count: navyShips?.kobaya || 0,
                              icon: "🦅",
                            },
                            {
                              id: "sekibune",
                              name: "関船 (Sekibune)",
                              desc: "水軍を力強く支える主力軍船。船首に大筒・鉄楯を備え、合戦時に凄まじい大筒砲撃【兵力加算: +350】の戦力加勢をもたらします。",
                              costGold: 800,
                              costCopper: 4000,
                              count: navyShips?.sekibune || 0,
                              icon: "🐉",
                            },
                            {
                              id: "atakebune",
                              name: "安宅船 (Atakebune)",
                              desc: "巨大な城郭風の総櫓を浮かせた戦海の超絶移動巨大要塞。周囲を威圧し、戦陣にて【兵力加算: +1,200】の爆発的破砕砲火を加勢します。",
                              costGold: 2500,
                              costCopper: 10000,
                              count: navyShips?.atakebune || 0,
                              icon: "⛩️",
                            },
                          ].map((ship) => {
                            const canAfford =
                              gold >= ship.costGold &&
                              copper >= ship.costCopper;
                            return (
                              <div
                                key={ship.id}
                                className="bg-dark-bg/90 border border-white/5 p-4 rounded flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 font-sans"
                              >
                                <div className="flex-1">
                                  <div className="flex items-center gap-2 mb-1">
                                    <span className="text-lg">{ship.icon}</span>
                                    <h4 className="text-xs font-serif font-bold text-stone-200">
                                      {ship.name}
                                    </h4>
                                    <span className="bg-cyan-950 text-cyan-400 font-mono text-[10px] px-2 py-0.5 rounded border border-cyan-800">
                                      就役: {ship.count} 隻
                                    </span>
                                  </div>
                                  <p className="text-[10px] text-stone-500 leading-relaxed mb-2 font-sans">
                                    {ship.desc}
                                  </p>
                                  <div className="flex gap-4 text-[9px] font-mono text-stone-500">
                                    <span
                                      className={
                                        gold >= ship.costGold
                                          ? "text-amber-500 font-semibold"
                                          : "text-stone-700"
                                      }
                                    >
                                      小判: -{ship.costGold}
                                    </span>
                                    <span
                                      className={
                                        copper >= ship.costCopper
                                          ? "text-amber-600 font-semibold"
                                          : "text-stone-700"
                                      }
                                    >
                                      銅銭: -{ship.costCopper}
                                    </span>
                                  </div>
                                </div>

                                <button
                                  onClick={() => {
                                    if (canAfford) {
                                      setGold((prev) => prev - ship.costGold);
                                      setCopper(
                                        (prev) => prev - ship.costCopper,
                                      );
                                      setNavyShips((prev) => ({
                                        ...prev,
                                        [ship.id]: (prev[ship.id] || 0) + 1,
                                      }));
                                      addLog(
                                        `【軍船進水】新たに ${ship.name} 1隻が進水、我が日本丸水軍艦隊に合流いたしました！`,
                                        "success",
                                      );
                                    } else {
                                      addLog(
                                        "【資源不足】国庫金員が足りず、造船起工が出来ません。",
                                        "error",
                                      );
                                    }
                                  }}
                                  className={`px-4 py-2 text-[10px] sm:text-xs font-serif font-bold rounded-sm transition-all border shrink-0 select-none ${
                                    canAfford
                                      ? "bg-cyan-950 border-cyan-500 text-cyan-400 hover:bg-cyan-400 hover:text-black hover:border-cyan-400"
                                      : "bg-stone-900 border-white/5 text-stone-700 cursor-not-allowed"
                                  }`}
                                >
                                  進水させる
                                </button>
                              </div>
                            );
                          })}
                        </div>
                      </div>

                      {/* 港湾造成 */}
                      <div className="bg-[#1c1917]/40 border border-white/5 p-6 rounded-sm flex flex-col">
                        <h3 className="font-serif font-bold text-lg text-cyan-300 mb-2 pb-2 border-b border-white/5 flex items-center gap-2">
                          <Anchor className="w-5 h-5 text-cyan-400" />{" "}
                          港湾造成と格式等級 (Coastal Ports)
                        </h3>
                        <p className="text-xs text-stone-500 mb-6 leading-relaxed font-sans font-sans">
                          支配下にある海岸（港の能力を持つ領土）の掘削と桟橋・倉庫整備を展開。港湾格式等級が高まると、徴税を行うたびに小判・銅銭が交易物流税として国庫に追加納入されます。
                        </p>

                        <div className="space-y-4 flex-1 overflow-y-auto custom-scrollbar max-h-[460px] pr-2">
                          {lands.filter(
                            (l) => l.isPort && occupiedLands.includes(l.id),
                          ).length === 0 ? (
                            <div className="text-center py-12 text-stone-600 font-sans text-xs italic leading-relaxed border border-dashed border-stone-800 p-6 rounded-sm bg-stone-950/20 max-w-lg mx-auto">
                              現在、海岸地域（港湾適地）を支配しておりません。
                              <br />
                              合戦にて、マップ上で水色の⚓マークがある沿岸領地（「薩摩」「肥前」「相模」「摂津」等）を攻め落とし、支配下に置くことで港湾造成が開始できます。
                            </div>
                          ) : (
                            lands
                              .filter(
                                (l) => l.isPort && occupiedLands.includes(l.id),
                              )
                              .map((land) => {
                                const harborLvl = land.harborLevel || 0;
                                const isMaxLvl = harborLvl >= 5;
                                const buildCostGold = 1000 + harborLvl * 500;
                                const buildCostCopper = 5000 + harborLvl * 2500;
                                const canUpgrade =
                                  gold >= buildCostGold &&
                                  copper >= buildCostCopper &&
                                  !isMaxLvl;

                                return (
                                  <div
                                    key={land.id}
                                    className="bg-dark-bg/90 border border-white/5 p-4 rounded flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4"
                                  >
                                    <div className="flex-1">
                                      <div className="flex items-center gap-2 mb-1.5">
                                        <span className="text-cyan-400 font-sans font-sans">
                                          ⚓
                                        </span>
                                        <h4 className="text-xs font-serif font-bold text-stone-200">
                                          {land.name}
                                        </h4>
                                        <span
                                          className={`text-[9px] px-2 py-0.5 rounded border font-mono ${
                                            harborLvl === 0
                                              ? "bg-stone-900 border-stone-800 text-stone-600"
                                              : "bg-cyan-950/80 border-cyan-800 text-cyan-400"
                                          }`}
                                        >
                                          格式等級:{" "}
                                          {harborLvl === 0
                                            ? "未開港"
                                            : `等級 壱・${harborLvl}`}
                                        </span>
                                      </div>
                                      <div className="text-[10px] text-stone-500 leading-normal space-y-1 my-2">
                                        <span className="text-emerald-400 font-sans font-semibold block">
                                          毎徴税時交易果:{" "}
                                          {harborLvl === 0
                                            ? "未稼働"
                                            : `小判 +${(harborLvl * 100).toLocaleString()}枚 / 銅銭 +${(harborLvl * 1200).toLocaleString()}貫`}
                                        </span>
                                      </div>
                                      {!isMaxLvl && (
                                        <div className="flex gap-4 text-[9px] font-mono text-stone-500 pt-1 border-t border-stone-800/40">
                                          <span>
                                            必要小判: -
                                            {buildCostGold.toLocaleString()}
                                          </span>
                                          <span>
                                            必要銅銭: -
                                            {buildCostCopper.toLocaleString()}
                                          </span>
                                        </div>
                                      )}
                                    </div>

                                    <button
                                      disabled={isMaxLvl}
                                      onClick={() => {
                                        if (canUpgrade) {
                                          setGold(
                                            (prev) => prev - buildCostGold,
                                          );
                                          setCopper(
                                            (prev) => prev - buildCostCopper,
                                          );

                                          const updatedLands = lands.map(
                                            (l) => {
                                              if (l.id === land.id) {
                                                return {
                                                  ...l,
                                                  harborLevel:
                                                    (l.harborLevel || 0) + 1,
                                                };
                                              }
                                              return l;
                                            },
                                          );
                                          setLands(updatedLands);
                                          addLog(
                                            `【港湾造成】${land.name} の造成を拡大し、海港格式等級を ${harborLvl + 1} に高めました！`,
                                            "success",
                                          );
                                        } else if (!isMaxLvl) {
                                          addLog(
                                            "【造成失敗】港湾造成に必要な国庫の資金が不足しております。",
                                            "error",
                                          );
                                        }
                                      }}
                                      className={`px-4 py-2 text-[10px] sm:text-xs font-serif font-bold rounded-sm transition-all border shrink-0 select-none ${
                                        isMaxLvl
                                          ? "bg-amber-950/20 border-amber-900/20 text-amber-500 cursor-default animate-pulse"
                                          : canUpgrade
                                            ? "bg-cyan-950 border-cyan-500 text-cyan-400 hover:bg-cyan-400 hover:text-black hover:border-cyan-400"
                                            : "bg-[#292524] border-white/5 text-stone-500 cursor-not-allowed"
                                      }`}
                                    >
                                      {harborLvl === 0
                                        ? "開港・造成"
                                        : isMaxLvl
                                          ? "最大格式"
                                          : "等級向上"}
                                    </button>
                                  </div>
                                );
                              })
                          )}
                        </div>
                      </div>
                    </div>
                  </div>
                )}
              </motion.div>
            )}

            {activeTab === "pvp" && (
              <motion.div
                key="pvp"
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="h-full flex flex-col bg-[#1c1917] rounded-sm border border-white/5 p-8 sm:p-12 mb-6"
              >
                {!user ? (
                  <div className="flex-1 border border-white/5 bg-black/40 rounded-sm p-12 flex flex-col items-center justify-center text-center">
                    <Sword className="w-20 h-20 text-red-500/30 mb-6 animate-pulse" />
                    <h3 className="text-xl font-serif font-bold text-stone-300 mb-4">
                      オンライン合戦にはログインが必要です
                    </h3>
                    <p className="text-xs text-stone-500 max-w-md leading-relaxed font-sans mb-8">
                      Googleアカウントで安全にログインすることで、自軍の編制をクラウドと同期し、全国のプレイヤーと真剣勝負ができるようになります。
                    </p>
                    <button
                      onClick={handleGoogleSignIn}
                      className="bg-red-500 hover:bg-red-400 text-white font-bold px-8 py-4 text-xs uppercase tracking-[0.2em] shadow-[0_0_20px_rgba(239,68,68,0.4)] transition-all rounded-sm"
                    >
                      Googleでログインする
                    </button>
                  </div>
                ) : !pvpActiveRoom ? (
                  <div className="flex-1 flex flex-col space-y-8 overflow-y-auto custom-scrollbar pr-2 p-1">
                    {/* Setup / Lobby Upper Controls */}
                    <div className="flex flex-col lg:flex-row justify-between items-start gap-8 border-b border-white/5 pb-8">
                      <div className="space-y-2">
                        <h2 className="text-4xl font-serif font-bold text-red-400 flex items-center gap-4 tracking-widest uppercase">
                          <Sword className="w-8 h-8 text-red-400" />{" "}
                          オンライン対戦場
                        </h2>
                        <p className="text-stone-500 font-sans text-sm leading-relaxed max-w-3xl">
                          誇り高き部隊を率いていざ、出陣！全国の猛将たちとリアルタイムで直接対決することができます。互いの采配（突撃・奇襲・堅守・火計）の読み合いが命運を分けます。
                        </p>
                      </div>

                      <div className="bg-[#1c1917]/80 border border-white/5 p-4 rounded-sm w-full lg:max-w-md space-y-4">
                        <h3 className="text-xs font-bold text-gold tracking-widest uppercase font-serif">武将の通り名・編成設定</h3>
                        <div className="flex gap-4">
                          <input
                            type="text"
                            value={pvpCustomName}
                            onChange={(e) => setPvpCustomName(e.target.value)}
                            placeholder={user.displayName || "殿の将名"}
                            maxLength={15}
                            className="w-full bg-black/60 border border-white/10 px-4 py-2 text-white outline-none focus:border-red-400 transition-colors font-sans text-xs"
                          />
                        </div>
                        <div className="space-y-1">
                          <label className="text-[10px] text-stone-500 font-bold uppercase tracking-wider block">参戦部隊の選択</label>
                          <div className="grid grid-cols-3 gap-2">
                            {[0, 1, 2].map((idx) => {
                              const power = calculateDeckPower(idx);
                              return (
                                <button
                                  key={idx}
                                  onClick={() => setPvpSelectedDeckIdx(idx)}
                                  className={`p-2 rounded border text-left transition-all ${
                                    pvpSelectedDeckIdx === idx
                                      ? "border-red-400 bg-red-400/10 text-white font-bold"
                                      : "border-white/5 bg-black/40 text-stone-500 hover:border-white/10"
                                  }`}
                                >
                                  <div className="text-[9px] text-stone-500">第{idx + 1}部隊</div>
                                  <div className="text-xs font-mono">{power.toLocaleString()}</div>
                                </button>
                              );
                            })}
                          </div>
                        </div>
                      </div>
                    </div>

                    {/* Room Creation & Lobby Rooms List Grid */}
                    <div className="grid grid-cols-1 xl:grid-cols-12 gap-8 flex-1">
                      {/* Left: Create Room Form */}
                      <div className="xl:col-span-4 bg-black/20 border border-white/5 p-6 rounded-sm flex flex-col justify-between h-fit space-y-6">
                        <div className="space-y-6">
                          <div>
                            <h3 className="text-md font-serif font-bold text-stone-300 mb-4 flex items-center gap-2">
                              <span className="w-1.5 h-6 bg-red-500 rounded-sm"></span>
                              合戦場を新設する
                            </h3>
                            <div className="space-y-4">
                              <div>
                                <label className="text-[10px] text-stone-500 font-bold uppercase tracking-wider block mb-1">
                                  合戦部屋名
                                </label>
                                <input
                                  type="text"
                                  value={pvpRoomNameInput}
                                  onChange={(e) => setPvpRoomNameInput(e.target.value)}
                                  placeholder="いざ尋常に勝負！初心者歓迎"
                                  maxLength={30}
                                  className="w-full bg-black/60 border border-white/10 px-4 py-3 text-white outline-none focus:border-red-400 transition-colors font-sans text-xs"
                                />
                              </div>
                            </div>
                          </div>

                          <button
                            onClick={handleCreatePvpRoom}
                            disabled={pvpMatchmaking}
                            className="w-full bg-red-500 hover:bg-red-400 disabled:opacity-30 text-white font-bold py-4 text-xs uppercase tracking-[0.2em] transition-all rounded-sm shadow-[0_0_20px_rgba(239,68,68,0.2)]"
                          >
                            {pvpMatchmaking ? "部屋起立中..." : "合戦場を新設"}
                          </button>

                          <div className="flex items-center gap-2 text-stone-500 my-1 justify-center">
                            <div className="flex-1 h-px bg-white/5 max-w-[40px]"></div>
                            <span className="text-[10px] font-sans uppercase tracking-widest text-[#a8a29e]">または</span>
                            <div className="flex-1 h-px bg-white/5 max-w-[40px]"></div>
                          </div>

                          <button
                            onClick={handleStartCpuBattle}
                            disabled={pvpMatchmaking}
                            className="w-full bg-stone-900 hover:bg-stone-800 disabled:opacity-30 text-stone-300 border border-white/10 font-bold py-3.5 text-xs uppercase tracking-[0.2em] transition-all rounded-sm"
                          >
                            仮想敵将と模擬戦 (CPU対戦)
                          </button>

                          <div className="border-t border-white/5 pt-6 space-y-4">
                            <h3 className="text-md font-serif font-bold text-stone-300 flex items-center gap-2">
                              <span className="w-1.5 h-6 bg-cyan-500 rounded-sm"></span>
                              部屋コードで参戦
                            </h3>
                            <div>
                              <label className="text-[10px] text-stone-500 font-bold uppercase tracking-wider block mb-1">
                                7桁の部屋コードを入力
                              </label>
                              <input
                                type="text"
                                value={pvpJoinCodeInput}
                                onChange={(e) => setPvpJoinCodeInput(e.target.value.replace(/\D/g, ""))}
                                placeholder="例: 1234567"
                                maxLength={7}
                                className="w-full bg-black/60 border border-white/10 px-4 py-3 text-white outline-none focus:border-cyan-400 transition-colors font-sans text-xs text-center font-mono tracking-widest text-lg"
                              />
                            </div>
                            <button
                              onClick={handleJoinByCode}
                              disabled={pvpMatchmaking || pvpJoinCodeInput.length !== 7}
                              className="w-full bg-cyan-600 hover:bg-cyan-500 disabled:opacity-30 text-white font-bold py-4 text-xs uppercase tracking-[0.2em] transition-all rounded-sm shadow-[0_0_20px_rgba(6,182,212,0.1)]"
                            >
                              参戦する
                            </button>
                          </div>
                        </div>
                      </div>

                      {/* Right: Active Rooms Grid */}
                      <div className="xl:col-span-8 space-y-4">
                        <div className="flex justify-between items-center bg-black/30 px-4 py-3 border border-white/5 rounded-sm">
                          <span className="text-xs font-serif text-stone-400 tracking-wider font-bold">参戦を待つ戦場一覧</span>
                          <span className="text-[10px] font-mono text-stone-600 uppercase tracking-widest block">
                            ● {pvpRoomsList.length} 陣地建設中
                          </span>
                        </div>

                        {pvpRoomsList.length === 0 ? (
                          <div className="border border-dashed border-white/5 bg-black/20 rounded-sm p-12 flex flex-col items-center justify-center text-center">
                            <RefreshCw className="w-12 h-12 text-stone-700 mb-4 animate-spin" style={{ animationDuration: '3s' }} />
                            <p className="text-xs text-stone-600 leading-relaxed font-sans max-w-sm">
                              現在対戦を募っている陣地が存在しません。あなたの一手で新設（ホスト開始）されるか、対戦相手をお待ちください。
                            </p>
                          </div>
                        ) : (
                          <div className="grid grid-cols-1 md:grid-cols-2 gap-4 max-h-[500px] overflow-y-auto custom-scrollbar pr-2">
                            {pvpRoomsList.map((room) => {
                              const isMyRoom = room.hostUid === user.uid;
                              const canJoin = room.status === "waiting" && !isMyRoom;
                              return (
                                <div
                                  key={room.roomId}
                                  className={`bg-black/40 border p-5 rounded-sm flex flex-col justify-between shadow-lg relative overflow-hidden transition-all hover:bg-white/5 ${
                                    isMyRoom ? "border-amber-500/30" : "border-white/5"
                                  }`}
                                >
                                  <div>
                                    <div className="flex justify-between items-start mb-2">
                                      <span className="text-[9px] font-mono text-stone-500 uppercase tracking-wider block">
                                        ROOM ID: {room.roomId}
                                      </span>
                                      <span
                                        className={`px-2 py-0.5 rounded text-[9px] font-bold tracking-wider ${
                                          room.status === "waiting"
                                            ? "bg-red-500/10 text-red-400 border border-red-500/20"
                                            : "bg-stone-500/10 text-stone-400 border border-white/5"
                                        }`}
                                      >
                                        {room.status === "waiting" ? "参戦募集中" : "合戦中"}
                                      </span>
                                    </div>
                                    <h4 className="text-sm font-serif font-bold text-white mb-3">
                                      {room.roomName}
                                    </h4>

                                    <div className="space-y-1 block text-[11px] text-stone-400 bg-black/20 p-3 rounded mb-4">
                                      <div className="flex justify-between">
                                        <span>将師: <strong className="text-stone-200">{room.hostName}</strong></span>
                                        <span className="text-gold font-mono">戦力: {room.hostPower.toLocaleString()}</span>
                                      </div>
                                      <div className="text-[10px] text-stone-500 mt-1 truncate">
                                        武将陣: {room.hostGenerals?.map((g: any) => g.name).join(", ")}
                                      </div>
                                    </div>
                                  </div>

                                  {isMyRoom ? (
                                    <div className="flex gap-2">
                                      <button
                                        onClick={() => subscribeToRoom(room.roomId)}
                                        className="flex-1 bg-amber-500 hover:bg-amber-400 text-black font-bold h-10 text-[10px] tracking-wider uppercase transition-all rounded-sm flex items-center justify-center animate-pulse"
                                      >
                                        自分の部屋に入る
                                      </button>
                                      <button
                                        onClick={() => handleClosePvpRoom(room.roomId)}
                                        className="bg-stone-800 hover:bg-stone-700 text-stone-300 h-10 px-3 text-[10px] transition-all rounded-sm"
                                        title="部屋を削除"
                                      >
                                        撤収
                                      </button>
                                    </div>
                                  ) : (
                                    <button
                                      onClick={() => handleJoinPvpRoom(room)}
                                      disabled={!canJoin || pvpMatchmaking}
                                      className={`w-full h-10 text-[10px] font-bold tracking-wider uppercase transition-all rounded-sm flex items-center justify-center ${
                                        canJoin
                                          ? "bg-red-500 hover:bg-red-400 text-white"
                                          : "bg-[#292524] border-white/5 text-stone-600 cursor-not-allowed"
                                      }`}
                                    >
                                      {room.status === "waiting" ? "この陣営に参戦" : "満員・合戦中"}
                                    </button>
                                  )}
                                </div>
                              );
                            })}
                          </div>
                        )}
                      </div>
                    </div>
                  </div>
                ) : (
                  /* Active Battle Game Arena screen */
                  <div className="flex-1 flex flex-col space-y-6 overflow-y-auto custom-scrollbar">
                    {/* Header bar */}
                    <div className="bg-black/60 border border-white/5 px-6 py-4 rounded-sm flex justify-between items-center shrink-0">
                      <div>
                        <span className="text-[10px] font-mono text-red-400 tracking-widest block uppercase font-bold">
                          ● 合戦領域 REALTIME BATTLE FIELD
                        </span>
                        <h3 className="text-md font-serif font-bold text-stone-200">
                          {pvpActiveRoom.roomName}
                        </h3>
                      </div>
                      
                      <button
                        onClick={() => {
                          if (pvpActiveRoom.hostUid === user.uid) {
                            handleClosePvpRoom(pvpActiveRoom.roomId);
                          } else {
                            handleLeavePvpRoom(pvpActiveRoom);
                          }
                        }}
                        className="bg-stone-900 border border-white/10 text-stone-400 hover:text-white px-4 py-2 rounded-sm text-xs transition-colors"
                      >
                        {pvpActiveRoom.hostUid === user.uid ? "陣地解散" : "対戦退却"}
                      </button>
                    </div>

                    {pvpActiveRoom.status === "waiting" ? (
                      /* A: Waiting for guest to join */
                      <div className="flex-1 border border-white/5 bg-black/40 rounded-sm p-6 lg:p-10 flex flex-col items-center">
                        <div className="flex items-center gap-3 mb-4">
                          <RefreshCw className="w-6 h-6 text-amber-500 animate-spin" style={{ animationDuration: '4s' }} />
                          <h4 className="text-md lg:text-lg font-serif font-bold text-amber-400">
                            対戦相手の参戦を待っています...
                          </h4>
                        </div>
                        <p className="text-xs text-stone-500 max-w-lg mb-8 text-center leading-relaxed">
                          他の武家・プレイヤーが戦場ロビーから参戦するのを待っています。こちらの【部屋コード（7桁）】を伝えて直接参戦してもらうこともできます。
                        </p>

                        <div className="grid grid-cols-1 md:grid-cols-2 gap-6 w-full max-w-3xl mb-6">
                          {/* Room Info & Code */}
                          <div className="bg-black/80 p-6 rounded-md border border-white/10 space-y-5 flex flex-col justify-between">
                            <div>
                              <div className="flex justify-between items-center text-stone-400 text-[10px] mb-2 border-b border-white/5 pb-2">
                                <span>合戦陣屋名</span>
                                <span className="text-stone-200 font-sans font-bold">{pvpActiveRoom.roomName}</span>
                              </div>
                              <div className="pt-2">
                                <label className="text-[10px] text-stone-400 uppercase tracking-wider block mb-1">部屋コード (7桁)</label>
                                <div className="text-3xl font-bold text-amber-400 tracking-widest font-mono bg-black px-4 py-3 rounded border border-amber-500/20 text-center select-all cursor-pointer hover:border-amber-500/40 transition-colors">
                                  {pvpActiveRoom.roomId}
                                </div>
                              </div>
                            </div>
                            
                            <div className="border-t border-white/5 pt-4 space-y-2 text-center">
                              <span className="text-stone-500 text-[10px] block select-none">相手がなかなか現れない場合、模擬戦を始められます</span>
                              <button
                                onClick={() => handleInjectCpuGuest(pvpActiveRoom)}
                                disabled={pvpMatchmaking}
                                className="w-full bg-amber-500 hover:bg-amber-400 text-black font-bold py-2.5 rounded-sm text-xs tracking-wider transition-all disabled:opacity-30"
                              >
                                {pvpMatchmaking ? "召喚中..." : "仮想敵将（CPU）を召喚して模擬戦開始"}
                              </button>
                            </div>
                          </div>

                          {/* Member List */}
                          <div className="bg-black/60 p-6 rounded-md border border-white/10 space-y-4">
                            <h5 className="text-xs font-serif font-bold text-stone-300 border-b border-white/10 pb-2 flex items-center justify-between">
                              <span>参戦中の武家一同</span>
                              <span className="text-[10px] text-stone-500 font-sans">2 / 2 枠空き</span>
                            </h5>
                            <div className="space-y-4">
                              {/* Host */}
                              <div className="space-y-1.5 text-left">
                                <div className="flex justify-between items-center">
                                  <span className="text-xs font-serif font-bold text-amber-300 flex items-center gap-1.5">
                                    <span className="w-2 h-2 rounded-full bg-amber-400 animate-pulse"></span>
                                    {pvpActiveRoom.hostName} (総大将)
                                  </span>
                                  <span className="text-[10px] text-stone-400 font-mono">戦力: {pvpActiveRoom.hostPower?.toLocaleString() || 0}</span>
                                </div>
                                {/* Host generals */}
                                <div className="flex flex-wrap gap-1 bg-black/40 p-2 rounded border border-white/5">
                                  {pvpActiveRoom.hostGenerals && pvpActiveRoom.hostGenerals.length > 0 ? (
                                    pvpActiveRoom.hostGenerals.map((g: any, i: number) => (
                                      <span key={i} className="text-[9px] bg-stone-900 border border-white/5 px-1.5 py-0.5 rounded text-stone-300">
                                        [{g.rarity}] {g.name} (Lv.{g.level})
                                      </span>
                                    ))
                                  ) : (
                                    <span className="text-[10px] text-stone-500">武将未編成</span>
                                  )}
                                </div>
                              </div>

                              {/* Guest */}
                              <div className="space-y-1.5 border-t border-white/5 pt-3 text-left">
                                <div className="flex justify-between items-center">
                                  <span className="text-xs font-serif text-stone-500 flex items-center gap-1.5">
                                    <span className="w-2 h-2 rounded-full bg-stone-700"></span>
                                    ゲスト参謀
                                  </span>
                                  <span className="text-[10px] text-stone-500 font-mono">対戦相手待機中...</span>
                                </div>
                                <div className="text-center py-4 bg-black/20 rounded border border-dashed border-white/10 text-[10px] text-stone-500">
                                  他プレイヤーが参戦すると自動で合戦が開始されます
                                </div>
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    ) : (
                      /* B/C: ACTIVE COMBAT OR CONGLUDED BATTLE */
                      <div className="flex-1 grid grid-cols-1 xl:grid-cols-12 gap-8 min-h-0 overflow-y-auto">
                        
                        {/* LEFT COLUMN: Players health and deck states (8/12 width) */}
                        <div className="xl:col-span-8 flex flex-col space-y-6 min-h-0">
                          
                          {/* Live Army Health Stand-off */}
                          <div className="grid grid-cols-2 gap-6 bg-black/30 border border-white/5 p-6 rounded-sm shrink-0">
                            {/* Host (Left Side) */}
                            <div className="space-y-3 border-r border-white/5 pr-4 text-left">
                              <div className="flex justify-between items-baseline">
                                <span className="text-xs font-serif font-bold text-stone-300">
                                  {pvpActiveRoom.hostName} {pvpActiveRoom.hostUid === user.uid && "(自分)"}
                                </span>
                                <span className="text-[10px] text-stone-500 font-mono uppercase">HOST</span>
                              </div>
                              <div className="h-4 bg-stone-900 border border-white/10 rounded-full overflow-hidden relative shadow-inner">
                                <motion.div
                                  className="h-full bg-gradient-to-r from-red-600 to-amber-500"
                                  animate={{ width: `${(pvpActiveRoom.hostHp / pvpActiveRoom.hostPower) * 100}%` }}
                                  transition={{ duration: 0.5 }}
                                />
                                <span className="absolute inset-0 flex items-center justify-center text-[10px] font-mono text-white text-shadow-sm font-bold">
                                  {pvpActiveRoom.hostHp.toLocaleString()} / {pvpActiveRoom.hostPower.toLocaleString()} HP
                                </span>
                              </div>
                              {/* Selection Indicator */}
                              <div className="flex items-center gap-2">
                                <span className="text-[9px] text-stone-500 uppercase font-mono font-bold">状況:</span>
                                {pvpActiveRoom.hostTactic ? (
                                  <span className="text-[10px] text-emerald-400 font-serif bg-emerald-950/40 border border-emerald-900/40 px-2 rounded-sm font-bold animate-pulse">● 采配決定（ロック済）</span>
                                ) : (
                                  <span className="text-[10px] text-stone-600 font-serif">思案中...</span>
                                )}
                              </div>
                            </div>

                            {/* Guest (Right Side) */}
                            <div className="space-y-3 pl-4 text-right">
                              <div className="flex justify-between items-baseline flex-row-reverse">
                                <span className="text-xs font-serif font-bold text-stone-300">
                                  {pvpActiveRoom.guestName} {pvpActiveRoom.guestUid === user.uid && "(自分)"}
                                </span>
                                <span className="text-[10px] text-stone-500 font-mono uppercase">GUEST</span>
                              </div>
                              <div className="h-4 bg-stone-900 border border-white/10 rounded-full overflow-hidden relative shadow-inner">
                                <motion.div
                                  className="h-full bg-gradient-to-r from-cyan-600 to-blue-500"
                                  animate={{ width: `${(pvpActiveRoom.guestHp / pvpActiveRoom.guestPower) * 100}%` }}
                                  transition={{ duration: 0.5 }}
                                />
                                <span className="absolute inset-0 flex items-center justify-center text-[10px] font-mono text-white text-shadow-sm font-bold">
                                  {pvpActiveRoom.guestHp.toLocaleString()} / {pvpActiveRoom.guestPower.toLocaleString()} HP
                                </span>
                              </div>
                              {/* Selection Indicator */}
                              <div className="flex items-center gap-2 justify-end">
                                <span className="text-[9px] text-stone-500 uppercase font-mono font-bold">状況:</span>
                                {pvpActiveRoom.guestTactic ? (
                                  <span className="text-[10px] text-cyan-400 font-serif bg-cyan-950/40 border border-cyan-900/40 px-2 rounded-sm font-bold animate-pulse">● 采配決定（ロック済）</span>
                                ) : (
                                  <span className="text-[10px] text-stone-600 font-serif">思案中...</span>
                                )}
                              </div>
                            </div>
                          </div>

                          {/* Both Armies Rosters & Active Vanguards */}
                          {(() => {
                            const activeHostIndex = (pvpActiveRoom.turn - 1) % (pvpActiveRoom.hostGenerals?.length || 3);
                            const activeGuestIndex = (pvpActiveRoom.turn - 1) % (pvpActiveRoom.guestGenerals?.length || 3);
                            return (
                              <div className="grid grid-cols-1 md:grid-cols-2 gap-4 bg-black/30 border border-white/5 p-4 rounded-sm shrink-0">
                                {/* Host Generals */}
                                <div className="space-y-2 text-left">
                                  <div className="flex justify-between items-center mb-1">
                                    <span className="text-[10px] text-stone-400 uppercase font-mono font-bold tracking-wider">
                                      ［東軍］武将布陣 (Host Forces)
                                    </span>
                                    <span className="text-[9px] text-red-400 font-serif">総戦力: {pvpActiveRoom.hostPower?.toLocaleString()}</span>
                                  </div>
                                  <div className="grid grid-cols-3 gap-2">
                                    {pvpActiveRoom.hostGenerals?.map((g: any, i: number) => {
                                      const isActive = i === activeHostIndex;
                                      const rarityColor = g.rarity === "SSR" ? "text-amber-400" :
                                                          g.rarity === "SR" ? "text-purple-400" :
                                                          "text-stone-400";
                                      return (
                                        <div
                                          key={i}
                                          className={`p-2 rounded border text-center transition-all flex flex-col justify-between relative ${
                                            isActive
                                              ? "bg-gradient-to-b from-red-950/30 to-black border-red-500/80 ring-1 ring-red-500/50 shadow-[0_0_10px_rgba(239,68,68,0.2)] scale-105"
                                              : "bg-black/20 border-white/5 opacity-50 text-stone-400"
                                          }`}
                                        >
                                          {isActive && (
                                            <span className="absolute -top-1.5 -left-1 px-1 bg-red-600 text-white font-bold text-[7px] rounded-sm uppercase tracking-wider animate-bounce">
                                              先陣
                                            </span>
                                          )}
                                          <div>
                                            <div className="flex justify-between items-center text-[8px] font-mono leading-none mb-1">
                                              <span className={`${rarityColor} font-bold`}>{g.rarity}</span>
                                              <span className="text-stone-500">Lv.{g.level}</span>
                                            </div>
                                            <div className="text-[10px] font-serif font-bold truncate text-stone-200">
                                              {g.name}
                                            </div>
                                          </div>
                                          
                                          <div className="grid grid-cols-3 gap-0.5 mt-2 text-[8px] font-mono border-t border-white/5 pt-1 text-stone-500">
                                            <div title="攻撃力">攻:{g.atk}</div>
                                            <div title="知力">知:{g.int}</div>
                                            <div title="防御力">防:{g.def}</div>
                                          </div>
                                        </div>
                                      );
                                    })}
                                  </div>
                                </div>

                                {/* Guest Generals */}
                                <div className="space-y-2 text-right">
                                  <div className="flex justify-between items-center flex-row-reverse mb-1">
                                    <span className="text-[10px] text-stone-400 uppercase font-mono font-bold tracking-wider">
                                      ［西軍］武将布陣 (Guest Forces)
                                    </span>
                                    <span className="text-[9px] text-cyan-400 font-serif">総戦力: {pvpActiveRoom.guestPower?.toLocaleString()}</span>
                                  </div>
                                  <div className="grid grid-cols-3 gap-2 text-left">
                                    {pvpActiveRoom.guestGenerals?.map((g: any, i: number) => {
                                      const isActive = i === activeGuestIndex;
                                      const rarityColor = g.rarity === "SSR" ? "text-amber-400" :
                                                          g.rarity === "SR" ? "text-purple-400" :
                                                          "text-stone-400";
                                      return (
                                        <div
                                          key={i}
                                          className={`p-2 rounded border text-center transition-all flex flex-col justify-between relative ${
                                            isActive
                                              ? "bg-gradient-to-b from-cyan-950/30 to-black border-cyan-500/80 ring-1 ring-cyan-500/50 shadow-[0_0_10px_rgba(6,182,212,0.2)] scale-105"
                                              : "bg-black/20 border-white/5 opacity-50 text-stone-400"
                                          }`}
                                        >
                                          {isActive && (
                                            <span className="absolute -top-1.5 -right-1 px-1 bg-cyan-600 text-white font-bold text-[7px] rounded-sm uppercase tracking-wider animate-bounce">
                                              先陣
                                            </span>
                                          )}
                                          <div>
                                            <div className="flex justify-between items-center text-[8px] font-mono leading-none mb-1">
                                              <span className={`${rarityColor} font-bold`}>{g.rarity}</span>
                                              <span className="text-stone-500">Lv.{g.level}</span>
                                            </div>
                                            <div className="text-[10px] font-serif font-bold truncate text-stone-200">
                                              {g.name}
                                            </div>
                                          </div>
                                          
                                          <div className="grid grid-cols-3 gap-0.5 mt-2 text-[8px] font-mono border-t border-white/5 pt-1 text-stone-500">
                                            <div title="攻撃力">攻:{g.atk}</div>
                                            <div title="知力">知:{g.int}</div>
                                            <div title="防御力">防:{g.def}</div>
                                          </div>
                                        </div>
                                      );
                                    })}
                                  </div>
                                </div>
                              </div>
                            );
                          })()}

                          {/* Tactics decisions panel or result view */}
                          {pvpActiveRoom.status === "ready" ? (
                            <div className="flex-1 flex flex-col justify-center bg-black/20 border border-white/5 p-6 rounded-sm space-y-4">
                              <div className="text-center space-y-2">
                                <span className="text-amber-500 font-mono text-lg font-bold">第 {pvpActiveRoom.turn} 合戦・軍配采配</span>
                                <p className="text-xs text-stone-500 max-w-md mx-auto">
                                  敵の予測を裏切る采配を選択せよ。突撃(Charge)は奇襲(Ambush)を制し、奇襲は火計(Fire)を、火計は堅守(Block)を、堅守は突撃を圧倒します！
                                </p>
                              </div>

                              {/* Interactive Tactical Affinity Guide Chart */}
                              <div className="bg-black/40 border border-white/5 rounded-md p-3 text-[11px] text-stone-400 flex flex-wrap items-center justify-around gap-2 max-w-2xl mx-auto w-full">
                                <span className="font-serif font-bold text-amber-500 text-xs tracking-wider">軍配三すくみ相関:</span>
                                <div className="flex items-center gap-1.5">
                                  <span className="text-red-400 font-bold bg-red-950/40 px-1.5 py-0.5 rounded border border-red-500/20 text-[10px]">突撃 ⚔️</span>
                                  <span className="text-stone-600 text-[10px]">＞</span>
                                  <span className="text-amber-400 font-bold bg-amber-950/40 px-1.5 py-0.5 rounded border border-amber-500/20 text-[10px]">奇襲 🦅</span>
                                </div>
                                <div className="flex items-center gap-1.5">
                                  <span className="text-amber-400 font-bold bg-amber-950/40 px-1.5 py-0.5 rounded border border-amber-500/20 text-[10px]">奇襲 🦅</span>
                                  <span className="text-stone-600 text-[10px]">＞</span>
                                  <span className="text-orange-400 font-bold bg-orange-950/40 px-1.5 py-0.5 rounded border border-orange-500/20 text-[10px]">火計 🔥</span>
                                </div>
                                <div className="flex items-center gap-1.5">
                                  <span className="text-orange-400 font-bold bg-orange-950/40 px-1.5 py-0.5 rounded border border-orange-500/20 text-[10px]">火計 🔥</span>
                                  <span className="text-stone-600 text-[10px]">＞</span>
                                  <span className="text-emerald-400 font-bold bg-emerald-950/40 px-1.5 py-0.5 rounded border border-emerald-500/20 text-[10px]">堅守 🛡️</span>
                                </div>
                                <div className="flex items-center gap-1.5">
                                  <span className="text-emerald-400 font-bold bg-emerald-950/40 px-1.5 py-0.5 rounded border border-emerald-500/20 text-[10px]">堅守 🛡️</span>
                                  <span className="text-stone-600 text-[10px]">＞</span>
                                  <span className="text-red-400 font-bold bg-red-950/40 px-1.5 py-0.5 rounded border border-red-500/20 text-[10px]">突撃 ⚔️</span>
                                </div>
                              </div>

                              {/* Action selection list */}
                              {((pvpActiveRoom.hostUid === user.uid && pvpActiveRoom.hostTactic) ||
                                (pvpActiveRoom.guestUid === user.uid && pvpActiveRoom.guestTactic)) ? (
                                <div className="border border-dashed border-white/5 bg-black/40 rounded-sm p-8 flex flex-col items-center justify-center text-center space-y-4">
                                  <div className="w-12 h-12 bg-white/5 rounded-full border border-white/10 flex items-center justify-center animate-spin">
                                    <RefreshCw className="w-5 h-5 text-red-500" />
                                  </div>
                                  <h4 className="text-sm font-serif font-bold text-stone-300">采配を定め、待命しております...</h4>
                                  <p className="text-[11px] text-stone-500 max-w-xs leading-relaxed">
                                    対戦相手が決断を下すことで、即座に合戦の幕が上がり、大激突が開始されます。
                                  </p>
                                  <button
                                    onClick={handleCancelReadyState}
                                    className="text-stone-500 hover:text-stone-300 text-xs border border-white/10 px-4 py-1.5 rounded transition-all mt-2"
                                  >
                                    采配を考え直す（取消）
                                  </button>
                                </div>
                              ) : (
                                <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
                                  {[
                                    { id: "charge", name: "突撃 (Charge)", sub: "対奇襲に大撃破", desc: "突撃 ⚔️", color: "border-red-500 text-red-400 bg-red-950/10 hover:bg-red-500/15 shadow-[0_0_15px_rgba(239,68,68,0.05)]" },
                                    { id: "ambush", name: "奇襲 (Ambush)", sub: "対火計を背後急襲", desc: "奇襲 🦅", color: "border-amber-500 text-amber-400 bg-amber-950/10 hover:bg-amber-500/15 shadow-[0_0_15px_rgba(245,158,11,0.05)]" },
                                    { id: "defend", name: "堅守 (Block)", sub: "対突撃を全盾防衛", desc: "堅守 🛡️", color: "border-emerald-500 text-emerald-400 bg-emerald-950/10 hover:bg-emerald-500/15 shadow-[0_0_15px_rgba(16,185,129,0.05)]" },
                                    { id: "fire", name: "火計 (Fire)", sub: "対堅守を灼熱熔解", desc: "火計 🔥", color: "border-orange-500 text-orange-400 bg-orange-950/10 hover:bg-orange-500/15 shadow-[0_0_15px_rgba(249,115,22,0.05)]" },
                                  ].map((t) => (
                                    <button
                                      key={t.id}
                                      onClick={() => handleSelectPvpTactic(t.id)}
                                      className={`p-4 rounded-sm border cursor-pointer border-opacity-30 flex flex-col justify-between items-center text-center transition-all min-h-[140px] select-none hover:scale-105 hover:border-opacity-100 ${t.color}`}
                                    >
                                      <div className="text-xl font-mono block opacity-80">{t.desc}</div>
                                      <div className="py-2">
                                        <div className="text-xs font-serif font-bold text-white tracking-widest">{t.name}</div>
                                        <div className="text-[9px] text-stone-500 mt-1 font-light">{t.sub}</div>
                                      </div>
                                    </button>
                                  ))}
                                </div>
                              )}
                            </div>
                          ) : pvpActiveRoom.status === "clashed" ? (
                            /* Clash Results screen showing comparison */
                            <div className="flex-1 flex flex-col justify-center bg-[#0d0706] border border-red-500/20 p-6 rounded-sm space-y-6 relative shadow-inner">
                              <div className="absolute top-4 left-6 text-[10px] text-red-500 font-mono tracking-widest font-bold">
                                ● 戦線激突決算 DIRECT CONFRONTATION
                              </div>

                              <div className="grid grid-cols-3 items-center text-center py-6">
                                <div className="space-y-2">
                                  <span className="text-[10px] text-stone-500 font-mono">HOST 采配</span>
                                  <div className="text-xl font-serif font-bold p-3 bg-red-950/40 border border-red-500/30 text-red-400 capitalize rounded inline-block min-w-[120px]">
                                    {pvpActiveRoom.hostTactic}
                                  </div>
                                </div>
                                <div className="text-stone-500 text-xl font-serif tracking-widest">VS</div>
                                <div className="space-y-2">
                                  <span className="text-[10px] text-stone-500 font-mono">GUEST 采配</span>
                                  <div className="text-xl font-serif font-bold p-3 bg-cyan-950/40 border border-cyan-500/30 text-cyan-400 capitalize rounded inline-block min-w-[120px]">
                                    {pvpActiveRoom.guestTactic}
                                  </div>
                                </div>
                              </div>

                              <div className="bg-black/60 border border-white/5 p-4 rounded text-center text-xs text-stone-300 leading-relaxed font-serif italic max-w-xl mx-auto">
                                {pvpActiveRoom.roundLogs[pvpActiveRoom.roundLogs.length - 1]}
                              </div>

                              <div className="flex justify-center pt-2">
                                <button
                                  onClick={async () => {
                                    const docRef = doc(db, "artifacts", APP_ID, "pvpRooms", pvpActiveRoom.roomId);
                                    let updatedTactic = { ...pvpActiveRoom, updatedAt: serverTimestamp() };
                                    if (pvpActiveRoom.hostUid === user.uid) {
                                      updatedTactic.hostTactic = "";
                                      if (pvpActiveRoom.guestUid === "cpu") {
                                        updatedTactic.guestTactic = "";
                                      }
                                    } else {
                                      updatedTactic.guestTactic = "";
                                    }
                                    await setDoc(docRef, updatedTactic);
                                    soundManager.playClick();
                                  }}
                                  className="bg-red-500 hover:bg-red-400 text-white font-bold px-10 py-3.5 text-xs uppercase tracking-[0.2em] rounded-sm transition-all"
                                >
                                  次の合戦へ名乗りを上げる
                                </button>
                              </div>
                            </div>
                          ) : (
                            /* Concluded screen */
                            <div className="flex-1 flex flex-col items-center justify-center bg-black/40 border border-white/5 p-12 rounded-sm text-center space-y-6">
                              {pvpActiveRoom.winnerUid === user.uid ? (
                                <div className="space-y-4">
                                  <Crown className="w-16 h-16 text-gold mx-auto animate-bounce" />
                                  <h4 className="text-4xl font-serif font-bold text-gold tracking-widest uppercase">
                                    天下御免の大勝利！
                                  </h4>
                                  <p className="text-xs text-stone-400 leading-relaxed max-w-sm mx-auto">
                                    あなたの圧倒的な武略が光りました。天下に名を轟かせる見事な采配でした！
                                  </p>
                                </div>
                              ) : !pvpActiveRoom.winnerUid ? (
                                <div className="space-y-4">
                                  <Activity className="w-16 h-16 text-stone-400 mx-auto" />
                                  <h4 className="text-4xl font-serif font-bold text-stone-300 tracking-widest uppercase">
                                    合戦、引き分け
                                  </h4>
                                  <p className="text-xs text-stone-500 leading-relaxed max-w-sm mx-auto">
                                    両陣営の優れた知謀が拮抗し、膠着の末引き分けに終わりました。
                                  </p>
                                </div>
                              ) : (
                                <div className="space-y-4">
                                  <Flag className="w-16 h-16 text-stone-700 mx-auto" />
                                  <h4 className="text-4xl font-serif font-bold text-stone-500 tracking-widest uppercase">
                                    我が軍 敗退...
                                  </h4>
                                  <p className="text-xs text-stone-500 leading-relaxed max-w-sm mx-auto">
                                    惜しくも敵陣に包囲され、撤退を余儀なくされました。鍛錬を重ね、再戦への牙を研ぎましょう。
                                  </p>
                                </div>
                              )}

                              <div className="pt-4">
                                <button
                                  onClick={() => {
                                    if (pvpActiveRoom.hostUid === user.uid) {
                                      handleClosePvpRoom(pvpActiveRoom.roomId);
                                    } else {
                                      handleLeavePvpRoom(pvpActiveRoom);
                                    }
                                  }}
                                  className="bg-stone-800 hover:bg-stone-700 text-white font-bold px-10 py-3.5 text-xs uppercase tracking-[0.2em] rounded-sm transition-all"
                                >
                                  合戦陣屋から退室
                                </button>
                              </div>
                            </div>
                          )}
                        </div>

                        {/* RIGHT COLUMN: Chat / Live Combat Logs (4/12 width) */}
                        <div className="xl:col-span-4 bg-black/40 border border-white/5 p-4 rounded-sm flex flex-col justify-between max-h-[550px] min-h-0 overflow-y-auto">
                          <h4 className="text-xs font-serif font-bold text-red-400 uppercase tracking-widest pb-3 border-b border-white/5 flex items-center gap-2 select-none">
                            <ScrollText className="w-4 h-4 text-red-500" />
                            本陣・合戦指令録
                          </h4>
                          
                          <div className="flex-1 overflow-y-auto space-y-4 py-4 pr-1 custom-scrollbar min-h-0">
                            {pvpActiveRoom.roundLogs?.map((log: string, idx: number) => (
                              <div
                                key={idx}
                                className={`text-[11px] font-sans leading-relaxed border-l-2 pl-3 py-1 ${
                                  log.includes("大勝利") || log.includes("勝者:")
                                    ? "border-amber-500 text-amber-200 bg-amber-500/5"
                                    : log.includes("参戦")
                                      ? "border-emerald-500 text-emerald-300"
                                      : "border-stone-800 text-stone-400"
                                }`}
                              >
                                {log}
                              </div>
                            ))}
                          </div>

                          <div className="text-[10px] text-stone-600 block uppercase font-mono border-t border-white/5 pt-3 leading-tight select-none">
                            合戦状況はリアルタイムで同期されています。
                          </div>
                        </div>

                      </div>
                    )}
                  </div>
                )}
              </motion.div>
            )}
          </AnimatePresence>
        </main>

        <aside className="w-96 bg-section-bg border-l border-white/5 flex flex-col">
          <div className="p-6 border-b border-white/5 flex items-center justify-between">
            <div className="flex items-center gap-3">
              <ScrollText className="w-4 h-4 text-gold/50" />
              <h3 className="font-bold text-[10px] uppercase tracking-[0.3em] text-stone-400">
                軍議録
              </h3>
            </div>
            <div className="flex gap-1">
              <div className="w-1 h-1 rounded-full bg-gold/30"></div>
              <div className="w-1 h-1 rounded-full bg-gold/50"></div>
              <div className="w-1 h-1 rounded-full bg-gold"></div>
            </div>
          </div>
          <div className="flex-1 overflow-y-auto p-8 space-y-6 custom-scrollbar text-xs leading-relaxed font-sans font-light">
            {logs.map((log) => (
              <div
                key={log.id}
                className={`animate-fade-in border-l-2 pl-4 py-1 transition-all ${
                  log.type === "rare"
                    ? "border-amber-500 text-amber-200 font-medium bg-amber-500/5 shadow-[0_0_10px_rgba(245,158,11,0.1)]"
                    : log.type === "error"
                      ? "border-red-600 text-red-500 bg-red-600/5"
                      : log.type === "success"
                        ? "border-emerald-500 text-emerald-400 bg-emerald-500/5"
                        : "border-white/10 text-stone-500"
                }`}
              >
                {log.text}
              </div>
            ))}
            <div ref={logsEndRef} />
          </div>
        </aside>
      </div>

      {/* Battle Scene Layer */}
      <AnimatePresence>
        {battleScene && (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            className="absolute inset-0 z-50 bg-black flex flex-col items-center justify-start p-6 md:p-12 overflow-y-auto"
          >
            {battleScene.phase === "intro" && (
              <motion.div
                initial={{ scale: 0.95, opacity: 0 }}
                animate={{ scale: 1, opacity: 1 }}
                className="max-w-4xl w-full mx-auto flex flex-col items-center justify-start py-8"
              >
                <h2 className="text-6xl font-serif font-bold text-gold mb-8 tracking-[0.3em] uppercase opacity-90 animate-pulse">
                  陣中軍議
                </h2>
                <div className="flex flex-col md:flex-row items-center gap-16 justify-center bg-stone-900/60 p-8 rounded border border-white/5 w-full mb-8">
                  <div className="text-center md:text-right flex-1">
                    <span className="block text-stone-500 uppercase text-[10px] mb-2 tracking-[0.3em] font-bold">
                      自軍総兵力
                    </span>
                    <span className="text-5xl font-mono text-cyan-300 tracking-tighter font-bold">
                      {battleScene.deckPower.toLocaleString()}
                    </span>
                    {battleScene.allianceReinforcements > 0 && (
                      <div className="text-[11px] text-emerald-400 font-sans mt-1">
                        (内、同盟軍援軍: +
                        {battleScene.allianceReinforcements.toLocaleString()})
                      </div>
                    )}
                    {battleScene.deckPower -
                      battleScene.baseDeckPower -
                      battleScene.allianceReinforcements >
                      0 && (
                      <div className="text-[11px] text-cyan-400 font-sans mt-0.5">
                        (内、水軍戦力加勢: +
                        {(
                          battleScene.deckPower -
                          battleScene.baseDeckPower -
                          battleScene.allianceReinforcements
                        ).toLocaleString()}
                        )
                      </div>
                    )}
                  </div>
                  <div className="text-3xl text-gold/30 font-serif italic uppercase tracking-widest">
                    VS
                  </div>
                  <div className="text-center md:text-left flex-1">
                    <span className="block text-stone-500 uppercase text-[10px] mb-2 tracking-[0.3em] font-bold">
                      敵軍：{battleScene.enemyName}
                    </span>
                    <span className="text-5xl font-mono text-red-400 tracking-tighter font-bold font-mono">
                      {battleScene.enemyPower.toLocaleString()}
                    </span>
                  </div>
                </div>

                {/* 合戦事態判定 (Battle Difficulty Indicator) */}
                {(() => {
                  const ratio = battleScene.deckPower / Math.max(1, battleScene.enemyPower);
                  let badgeColor = "bg-stone-900 border-white/5 text-stone-300";
                  let statusText = "実力伯仲";
                  let descText = "両陣の勢力は拮抗しておる。攻め落とすか、手引きされるか。軍略を尽くし敵を打破せよ！";

                  if (ratio >= 2.0) {
                    badgeColor = "bg-emerald-950 border-emerald-500/40 text-emerald-400";
                    statusText = "我軍圧倒（勝機到来）";
                    descText = "我が軍が圧倒的に優勢である！敵は籠城し恐れをなしておる。無血での一括降伏勧告（無血開城）が有効じゃ。";
                  } else if (ratio >= 1.25) {
                    badgeColor = "bg-cyan-950 border-cyan-500/40 text-cyan-400";
                    statusText = "幾分有利";
                    descText = "我が軍が幾分有利な態勢にある。包囲や奇襲などの定石を駆使すれば勝機は極めて高い。";
                  } else if (ratio <= 0.45) {
                    badgeColor = "bg-red-950/40 border-red-500 text-red-400 animate-pulse";
                    statusText = "絶望的劣勢（玉砕の恐れ）";
                    descText = "敵の勢力は圧倒的。通常正面から戦えば全軍玉砕は避けられぬ。命をかけた『背水の陣』で一か八かの奇跡に縋る他なし！";
                  } else if (ratio <= 0.8) {
                    badgeColor = "bg-amber-950/40 border-amber-500/40 text-amber-500";
                    statusText = "不利の兆候";
                    descText = "敵軍が優勢。このままでは苦しい戦いとなる。兵糧攻めで敵の戦闘力を削ぐか、法螺貝による陽動で崩せ。";
                  }

                  return (
                    <div className={`w-full max-w-4xl p-6 rounded-sm border mb-8 flex flex-col md:flex-row items-start md:items-center justify-between gap-4 font-sans ${ratio <= 0.45 ? "border-red-900 bg-[#161212]/90 shadow-[0_0_15px_rgba(239,68,68,0.15)]" : ratio >= 2.0 ? "border-emerald-800 bg-[#121614]/85" : "border-white/5 bg-[#141414]"}`}>
                      <div className="flex-1">
                        <div className="flex items-center gap-3 mb-2">
                          <span className={`px-2 py-0.5 text-[10px] font-bold font-serif tracking-widest uppercase border rounded-sm ${badgeColor}`}>
                            局勢：{statusText}
                          </span>
                          <span className="text-[10px] text-stone-500 font-mono">
                            戦力比率: {ratio.toFixed(2)}倍
                          </span>
                        </div>
                        <p className="text-xs text-stone-300 leading-relaxed font-serif">
                          {descText}
                        </p>
                      </div>
                    </div>
                  );
                })()}

                {/* 特殊支援（同盟軍 / 海軍）の合流詳細表示 */}
                {battleScene.reinforcementLogs &&
                  battleScene.reinforcementLogs.length > 0 && (
                    <div className="mb-8 w-full max-w-2xl bg-black/60 border border-cyan-500/10 p-4 rounded text-left">
                      <div className="text-cyan-400 font-serif font-bold text-xs tracking-widest flex items-center gap-2 mb-2">
                        <Sparkles className="w-3.5 h-3.5 text-cyan-400 animate-pulse" />{" "}
                        沿岸防禦・同盟軍集結完了
                      </div>
                      <div className="text-stone-400 text-[11px] font-sans space-y-1">
                        {battleScene.reinforcementLogs.map(
                          (logStr: string, idx: number) => (
                            <div key={idx} className="flex justify-between">
                              <span>{logStr}</span>
                            </div>
                          ),
                        )}
                      </div>
                    </div>
                  )}

                {/* 攻城作戦・軍略指示カード */}
                <div className="w-full max-w-3xl text-left bg-stone-900 border border-gold/20 p-6 rounded relative overflow-hidden">
                  <div className="absolute top-2 right-4 text-[9px] font-mono text-amber-500/40 uppercase tracking-widest">
                    WAR COUNCIL DECK
                  </div>
                  <h3 className="text-sm font-bold text-gold uppercase tracking-wider mb-6 flex items-center gap-2 font-serif border-b border-white/5 pb-2">
                    <Sword className="w-4 h-4 text-gold animate-bounce" />
                    本陣軍略選定 (Military Operations Council)
                  </h3>
                  <p className="text-[11px] text-stone-400 mb-6 font-sans leading-relaxed">
                    総大将、合戦に臨む陣形・作戦をお選びくだされ。備品を活用した特殊火器・陽動の奇襲、または国費を投じた兵糧攻めにより、形勢を強硬に好転させられます。
                  </p>

                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    {/* Tactic Override for Overwhelming Strength: Bloodless Surrender */}
                    {battleScene.deckPower / Math.max(1, battleScene.enemyPower) >= 2.0 && (
                      <button
                        onClick={() => launchTacticalSiege("surrender_immediate")}
                        className="text-left bg-emerald-950/20 hover:bg-emerald-950/40 p-4 rounded border-2 border-emerald-500/50 hover:border-emerald-400 transition-all group cursor-pointer col-span-1 md:col-span-2"
                      >
                        <div className="flex justify-between items-start mb-2">
                          <span className="font-serif font-bold text-sm text-emerald-300 group-hover:text-emerald-200 transition-colors flex items-center gap-2 animate-pulse">
                            <span>🏳️</span> 圧倒的勝機・一括無血開城勧告
                          </span>
                          <span className="text-[9px] bg-emerald-900 text-emerald-100 px-1.5 py-0.5 rounded leading-none font-bold">局勢限定</span>
                        </div>
                        <p className="text-[10px] text-emerald-200/70 leading-normal mb-3 font-sans">我が軍の圧倒的な兵力の威容を誇示し、使者を送って全面開城を勧告します。不必要な血を流さずに城を即座に占領します。</p>
                        <div className="flex justify-between items-center text-[10px] font-serif border-t border-emerald-800/30 pt-2">
                          <span className="text-emerald-400 font-bold">無血開城（戦闘を不戦大勝利で即座に決着）</span>
                          <span className="text-emerald-400 font-mono">費用: 無償</span>
                        </div>
                      </button>
                    )}

                    {/* Tactic Override for Desperate Disadvantage: Suicide Gambit */}
                    {battleScene.deckPower / Math.max(1, battleScene.enemyPower) <= 0.45 && (
                      <button
                        onClick={() => launchTacticalSiege("gambit_suicide")}
                        className="text-left bg-red-950/20 hover:bg-red-950/40 p-4 rounded border-2 border-red-500 hover:border-red-400 transition-all group cursor-pointer col-span-1 md:col-span-2 shadow-[0_0_12px_rgba(239,68,68,0.15)] animate-pulse"
                      >
                        <div className="flex justify-between items-start mb-2">
                          <span className="font-serif font-bold text-sm text-red-300 group-hover:text-red-200 transition-colors flex items-center gap-2">
                            <span>🎌</span> 乾坤一擲・決死背水の陣総起立
                          </span>
                          <span className="text-[9px] bg-red-900 text-red-100 px-1.5 py-0.5 rounded leading-none font-bold">逆転奇策</span>
                        </div>
                        <p className="text-[10px] text-red-200/85 leading-normal mb-3 font-sans">川を背にして退路を断ち「生きるか死ぬか」の極限状態を自ら演出。全員の精神的限界を超越した狂乱破壊力で挑みます！</p>
                        <div className="flex justify-between items-center text-[10px] font-serif border-t border-red-800/30 pt-2">
                          <span className="text-red-400 font-bold">自軍総戦闘力 +110% ＆ 奇跡的勝利で戦利品・銭貨が2倍！</span>
                          <span className="text-red-400 font-mono">費用: 無償</span>
                        </div>
                      </button>
                    )}

                    {/* Tactic 1: Assault */}
                    <button
                      onClick={() => launchTacticalSiege("assault")}
                      className="text-left bg-black/40 hover:bg-stone-800 p-4 rounded border border-white/5 hover:border-gold/30 transition-all group cursor-pointer"
                    >
                      <div className="flex justify-between items-start mb-2">
                        <span className="font-serif font-bold text-sm text-stone-200 group-hover:text-gold transition-colors flex items-center gap-2">
                          <span>⚔️</span> 正面突破・魚鱗の陣
                        </span>
                        <span className="text-[9px] bg-stone-800 text-stone-400 px-1.5 py-0.5 rounded leading-none">基本</span>
                      </div>
                      <p className="text-[10px] text-stone-500 leading-normal mb-3 font-sans">主力突撃隊を一列に集結させ難攻落城への強襲戦を慣行。正面から城門を打ち破ります。</p>
                      <div className="flex justify-between items-center text-[10px] font-serif border-t border-white/5 pt-2">
                        <span className="text-cyan-400 font-bold">自軍総兵力 +15% 強力化</span>
                        <span className="text-stone-400 font-mono">費用: 無償</span>
                      </div>
                    </button>

                    {/* Tactic 2: Crane */}
                    <button
                      onClick={() => launchTacticalSiege("crane")}
                      className="text-left bg-black/40 hover:bg-stone-800 p-4 rounded border border-white/5 hover:border-gold/30 transition-all group cursor-pointer"
                    >
                      <div className="flex justify-between items-start mb-2">
                        <span className="font-serif font-bold text-sm text-stone-200 group-hover:text-gold transition-colors flex items-center gap-2">
                          <span>🦅</span> 包囲殲滅・鶴翼の陣
                        </span>
                        <span className="text-[9px] bg-stone-800 text-stone-400 px-1.5 py-0.5 rounded leading-none">基本</span>
                      </div>
                      <p className="text-[10px] text-stone-500 leading-normal mb-3 font-sans">兵を左右に大きく鶴の翼の如く展開し城外殻を完全包囲。敵要塞の防衛連撃を封じます。</p>
                      <div className="flex justify-between items-center text-[10px] font-serif border-t border-white/5 pt-2">
                        <span className="text-cyan-400 font-bold">敵守備兵数 -10% 低下</span>
                        <span className="text-stone-400 font-mono">費用: 無償</span>
                      </div>
                    </button>

                    {/* Tactic 3: Horoku */}
                    <button
                      onClick={() => launchTacticalSiege("horoku")}
                      className="text-left bg-black/40 hover:bg-stone-800 p-4 rounded border border-white/5 hover:border-gold/30 transition-all group cursor-pointer"
                    >
                      <div className="flex justify-between items-start mb-2">
                        <span className="font-serif font-bold text-sm text-stone-200 group-hover:text-gold transition-colors flex items-center gap-2">
                          <span>🔥</span> 焙烙大筒・面制圧射撃
                        </span>
                        <span className="text-[9px] bg-amber-950/45 text-amber-400 border border-amber-900/30 px-1.5 py-0.5 rounded leading-none">特化</span>
                      </div>
                      <p className="text-[10px] text-stone-500 leading-normal mb-3 font-sans">
                        安宅船の焙烙弾・大筒砲術隊を総動員し天守へ流星雨。城壁ごと敵兵力抵抗を爆砕粉砕。
                      </p>
                      <div className="flex justify-between items-center text-[10px] font-serif border-t border-white/5 pt-2">
                        <span className="text-amber-400 font-bold">自軍総兵力 +35% 極大補正</span>
                        <span className="text-stone-400 font-mono">
                          {inventory.drum > 0 ? "陣太鼓を1個消費" : inventory.tactics > 0 ? "兵法書を1個消費" : "国費 3,000銭消費"}
                        </span>
                      </div>
                    </button>

                    {/* Tactic 4: Decoy */}
                    <button
                      onClick={() => launchTacticalSiege("decoy")}
                      className="text-left bg-black/40 hover:bg-stone-800 p-4 rounded border border-white/5 hover:border-gold/30 transition-all group cursor-pointer"
                    >
                      <div className="flex justify-between items-start mb-2">
                        <span className="font-serif font-bold text-sm text-stone-200 group-hover:text-gold transition-colors flex items-center gap-2">
                          <span>🐚</span> 陽動作戦・搦手虚実攻
                        </span>
                        <span className="text-[9px] bg-amber-950/45 text-amber-400 border border-amber-900/30 px-1.5 py-0.5 rounded leading-none">特化</span>
                      </div>
                      <p className="text-[10px] text-stone-500 leading-normal mb-3 font-sans">
                        囮の太鼓・法螺吹鳴で敵を引き付け、搦手口（裏門）より忍びの奇襲隊を侵入させ城内を錯乱。
                      </p>
                      <div className="flex justify-between items-center text-[10px] font-serif border-t border-white/5 pt-2">
                        <span className="text-amber-400 font-bold">自軍兵力 +25% ＆ 武具ドロップ向上</span>
                        <span className="text-stone-400 font-mono">
                          {inventory.hora > 0 ? "法螺貝を1個消費" : "国費 2,000銭消費"}
                        </span>
                      </div>
                    </button>
                  </div>

                  {/* Starvation Alternative */}
                  <div className="mt-4 pt-3 border-t border-stone-800 text-center">
                    <button
                      onClick={() => launchTacticalSiege("starve")}
                      className="text-[10px] font-serif text-stone-400 hover:text-red-400 underline tracking-wider cursor-pointer"
                    >
                      備品がない？ 国費1,200銅銭を投じて【水路遮断・兵糧攻め（敵守備力 -18%）】を実行する場合はここをクリック
                    </button>
                  </div>
                </div>
              </motion.div>
            )}

            {battleScene.phase === "clash" && (
              <div className="w-full max-w-4xl flex flex-col items-center justify-start py-8">
                {/* 攻城天守・城郭破壊進行度メーター */}
                <div className="w-full max-w-2xl text-center bg-stone-900 border border-cyan-500/20 p-6 rounded shadow-2xl mb-8 relative overflow-hidden">
                  <div className="absolute top-1 right-2 text-[8px] font-mono text-cyan-400/40">SIEGE WORK PROGRESS</div>
                  <div className="font-serif text-[11px] uppercase text-stone-400 tracking-widest mb-1.5 flex items-center justify-center gap-2">
                    <Anchor className="w-3.5 h-3.5 text-cyan-400 animate-spin" />
                    {battleScene.clashStep === 0 && "第一城郭外濠包囲網（攻城開始）"}
                    {battleScene.clashStep === 1 && "大手口巨大城門（破城槌激突中）"}
                    {battleScene.clashStep === 2 && "天守曲輪・二の丸（白兵戦展開中）"}
                    {battleScene.clashStep === 3 && "本丸大天守閣（頂上対決！）"}
                    {battleScene.clashStep >= 4 && "全戦線集計（勝敗の判定中）"}
                  </div>
                  
                  {/* Durability representation bar */}
                  <div className="w-full h-3.5 bg-black/60 rounded overflow-hidden border border-white/5 relative p-0.5">
                    <motion.div
                      initial={{ width: "100%" }}
                      animate={{
                        width:
                          battleScene.clashStep === 0
                            ? "100%"
                            : battleScene.clashStep === 1
                              ? "75%"
                              : battleScene.clashStep === 2
                                ? "40%"
                                : battleScene.clashStep === 3
                                  ? "15%"
                                  : "0%"
                      }}
                      className="h-full bg-gradient-to-r from-red-600 via-orange-500 to-cyan-400 rounded-sm"
                      style={{ transition: "width 1s cubic-bezier(0.16, 1, 0.3, 1)" }}
                    />
                  </div>
                  <div className="flex justify-between items-center text-[9px] font-mono text-stone-500 mt-1.5">
                    <span>城郭外濠 integrity: 100%</span>
                    <span className="text-cyan-300 font-bold">
                      城郭崩壊率:{" "}
                      {battleScene.clashStep === 0 && "0%"}
                      {battleScene.clashStep === 1 && "25%"}
                      {battleScene.clashStep === 2 && "60%"}
                      {battleScene.clashStep === 3 && "85%"}
                      {battleScene.clashStep >= 4 && "100%"}
                    </span>
                    <span>天守閣 Keep: 0%</span>
                  </div>
                </div>

                {/* Main animated charging screen */}
                <div className="w-full flex items-center justify-center overflow-hidden h-40 mb-8 relative">
                  <div className="absolute inset-x-0 bottom-0 top-auto h-[1px] bg-gradient-to-r from-transparent via-cyan-500/20 to-transparent"></div>
                  <div className="flex gap-40 z-10 items-center justify-center">
                    <div className="flex flex-col gap-3 items-end">
                      {battleScene.activeDeck.map((g: any, i: number) => (
                        <div
                          key={i}
                          className="animate-charge-right flex items-center gap-3 py-1.5 px-4 bg-white/5 border border-white/10 rounded-sm shadow-xl"
                          style={{ animationDelay: `${i * 0.1}s` }}
                        >
                          <span className="font-serif font-bold text-white text-xs tracking-wider">
                            {g.name}
                          </span>
                          <Sword className="w-4 h-4 text-gold shrink-0" />
                        </div>
                      ))}
                    </div>
                    <div className="flex items-center justify-center scale-125">
                      <div className="animate-shake relative">
                        <Sword className="w-16 h-16 text-gold animate-clash-sparks" />
                        <Shield className="w-16 h-16 text-red-900 absolute top-0 animate-clash-sparks-reverse opacity-30" />
                      </div>
                    </div>
                    <div className="flex flex-col gap-3 items-start">
                      {[1, 2, 3].map((i) => (
                        <div
                          key={i}
                          className="animate-charge-left flex items-center gap-3 py-1.5 px-4 bg-red-950/20 border border-red-900/30 rounded-sm shadow-xl"
                          style={{ animationDelay: `${i * 0.1}s` }}
                        >
                          <Shield className="w-4 h-4 text-red-800 shrink-0" />
                          <span className="font-serif font-bold text-stone-500 text-xs tracking-wider">
                            敵軍防衛隊
                          </span>
                        </div>
                      ))}
                    </div>
                  </div>
                </div>

                {/* Military Live Log Display feed */}
                <div className="w-full max-w-2xl bg-black/80 border border-gold/15 p-5 rounded font-sans relative mb-6">
                  <div className="absolute -top-2.5 left-4 bg-black px-2 text-[9px] font-serif font-bold text-gold tracking-widest border border-gold/15">
                     本陣強襲実況 (Battlefield Dispatch)
                  </div>
                  <div className="space-y-2.5 max-h-48 overflow-y-auto custom-scrollbar font-sans">
                    {battleScene.clashLogs &&
                      battleScene.clashLogs.map((log: string, idx: number) => (
                        <motion.div
                          key={idx}
                          initial={{ opacity: 0, x: -10 }}
                          animate={{ opacity: 1, x: 0 }}
                          className={`text-xs text-left leading-relaxed py-1 font-serif ${
                            idx === battleScene.clashLogs.length - 1
                              ? "text-gold font-bold bg-gold/5 px-2 rounded-sm border-l border-gold border-dashed"
                              : "text-stone-400 pl-2 border-l border-white/5"
                          }`}
                        >
                          {log}
                        </motion.div>
                      ))}
                  </div>
                </div>

                {/* Interactive Tactical Choices */}
                {battleScene.clashStep <= 3 && (
                  <div className="w-full max-w-2xl bg-[#1c1917]/90 border border-gold/30 p-6 rounded text-center shadow-[0_4px_35px_rgba(0,0,0,0.6)] animate-fade-in">
                    <div className="font-serif text-xs text-amber-500 uppercase tracking-[0.25em] mb-1 font-bold">
                      ⚔️ 指揮官戦闘采配 ⚔️
                    </div>
                    <h4 className="font-serif text-base text-stone-200 mb-3 font-semibold">
                      {battleScene.clashStep === 0 && "【第一関門】 攻城戦アプローチ：外門突破の指揮"}
                      {battleScene.clashStep === 1 && "【第一の丸】 要害攻防：一の丸突破の采配"}
                      {battleScene.clashStep === 2 && "【第二の丸】 戦線崩壊：二の丸・曲輪迂回襲撃"}
                      {battleScene.clashStep === 3 && "【天守閣決戦】 本丸本陣：敵守備総大将への猛襲決着"}
                    </h4>
                    <p className="text-[11px] text-stone-400 mb-5 leading-normal max-w-lg mx-auto">
                      {battleScene.clashStep === 0 && "堅牢なる外濠が立ちはだかります。敵の狙撃兵から猛烈な矢・鉄砲の雨が降り注ぐ中、どの突撃路を攻略しますか？"}
                      {battleScene.clashStep === 1 && "外濠の第一城門を突破し、主郭「一の丸」の攻撃に移ります。急な土塁・石落としの網をいかに対策して占拠しますか？"}
                      {battleScene.clashStep === 2 && "敵の防衛中枢「二の丸」に、多数の籠城守備隊が集結しております。正面抵抗を無力化し、本丸への道を切り開きましょう！"}
                      {battleScene.clashStep === 3 && "ついに敵総大将が布陣する本陣「本丸大天守閣」へと肉薄！最後の総攻撃、どう勝負を決めますか？"}
                    </p>

                    <div className="grid grid-cols-1 sm:grid-cols-3 gap-3">
                      {battleScene.clashStep === 0 && (
                        <>
                          <button
                            onClick={() => advanceClashDecision("front_gate")}
                            className="bg-red-950/40 hover:bg-red-900 border border-red-800 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-red-400">🚪 大手大門強襲</span>
                            破城槌で一気呵成に正面扉を打ち砕く！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 自軍戦闘力 +10%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("back_gate")}
                            className="bg-black/80 hover:bg-stone-800 border border-stone-800 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-amber-500">👣 裏門（搦手）急襲</span>
                            備えが極薄な裏口を別部隊で不意打ち！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 敵軍守備力 -10%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("moat_conduit")}
                            className="bg-[#052e16]/30 hover:bg-[#064e3b]/50 border border-teal-900 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-teal-400">💧 外濠水路潜入</span>
                            冷たい水路を潜水進軍し、闇から攀登！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 敵軍守備力 -8%</span>
                          </button>
                        </>
                      )}

                      {battleScene.clashStep === 1 && (
                        <>
                          <button
                            onClick={() => advanceClashDecision("first_ward")}
                            className="bg-red-950/40 hover:bg-red-900 border border-red-800 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-red-400">🛡️ 一の丸力押し突破</span>
                            全員抜刀！急傾を遮二無二に攀じ登り踏み潰す！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 自軍戦闘力 +10%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("dry_moat")}
                            className="bg-black/80 hover:bg-stone-800 border border-stone-800 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-amber-500">🧗 空堀急絶壁攀登</span>
                            縄梯子を用い、横合の崖から城壁を頭上急襲！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 敵軍守備力 -10%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("towers")}
                            className="bg-[#3b0764]/30 hover:bg-[#581c87]/50 border border-purple-900 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-purple-400">💥 櫓窓へ焙烙連射</span>
                            守備用の高楼へ焙烙玉と火矢を多重乱擲！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 自軍戦闘力 +8%</span>
                          </button>
                        </>
                      )}

                      {battleScene.clashStep === 2 && (
                        <>
                          <button
                            onClick={() => advanceClashDecision("second_ward_assault")}
                            className="bg-red-950/40 hover:bg-red-900 border border-red-800 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-red-400">⚡ 二の丸正面猛強襲</span>
                            千鳥口の大鉄門へ火縄銃の弾幕と共に直撃！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 自軍戦闘力 +12%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("ninja_infiltrate")}
                            className="bg-black/80 hover:bg-stone-800 border border-stone-800 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-teal-400">🕸️ 地下ぬけ穴忍侵入</span>
                            忍びを裏の勝手口へ放ち、内部から全郭爆破！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 敵軍守備力 -12%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("cry_decoy")}
                            className="bg-[#2a1305]/30 hover:bg-[#431407]/50 border border-orange-950 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-orange-400">🎺 時の声陽動大作戦</span>
                            猛烈な陣太鼓・法螺を他方で鳴らし敵を分散！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 敵軍守備力 -8%</span>
                          </button>
                        </>
                      )}

                      {battleScene.clashStep === 3 && (
                        <>
                          <button
                            onClick={() => advanceClashDecision("duel")}
                            className="bg-amber-950/60 hover:bg-amber-900 border border-amber-500 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif animate-pulse"
                          >
                            <span className="block font-bold mb-1 text-amber-400">⚔️ 一騎討ちを申し出る</span>
                            敵軍大将とのタイマン勝負で決着！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 自軍戦闘力 +15%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("flame_arrows")}
                            className="bg-red-950/40 hover:bg-red-900 border border-red-800 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-red-500">🏹 天守閣焦熱焼き討ち</span>
                            大天守に向けて総焙烙矢を射掛け、炎上陥落！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 敵軍守備力 -15%</span>
                          </button>
                          <button
                            onClick={() => advanceClashDecision("surrender")}
                            className="bg-stone-900 hover:bg-stone-800 border border-stone-850 text-stone-100 text-[11px] py-3 px-2 rounded hover:scale-105 active:scale-95 transition-all text-left font-serif"
                          >
                            <span className="block font-bold mb-1 text-stone-400">💬 本丸総包囲降伏勧告</span>
                            兵糧枯渇を盾に、降伏に追い込んで無血開城！
                            <span className="block font-mono text-[9px] text-cyan-400 mt-2 font-semibold">効果: 敵軍守備力 -12%</span>
                          </button>
                        </>
                      )}
                    </div>

                    <div className="flex justify-between items-center mt-6 pt-4 border-t border-white/5 text-[10px] font-mono text-stone-500">
                      <div>
                        我が軍実行兵力: <span className="text-cyan-400 font-bold">{(battleScene.deckPower || 0).toLocaleString()}</span>
                      </div>
                      <div>
                        敵城防衛戦力: <span className="text-red-400 font-bold">{(battleScene.enemyPower || 0).toLocaleString()}</span>
                      </div>
                    </div>
                  </div>
                )}

                {battleScene.clashStep >= 4 && (
                  <div className="w-full max-w-2xl bg-black/40 border border-white/5 py-8 rounded mt-4 text-center animate-pulse">
                    <p className="text-xs text-stone-400 font-serif">
                      総大将の御采配が完了いたしました！これより最終勝敗の戦決定戦が下されます...。
                    </p>
                  </div>
                )}
              </div>
            )}

            {battleScene.phase === "result" && (
              <motion.div
                initial={{ scale: 0.9, opacity: 0 }}
                animate={{ scale: 1, opacity: 1 }}
                className="text-center bg-[#121212] border border-gold/30 p-12 md:p-16 rounded-sm shadow-[0_0_100px_rgba(197,160,89,0.1)] relative max-w-3xl mx-auto w-full"
              >
                <div className="absolute top-4 left-4 opacity-10 font-serif text-8xl uppercase tracking-tighter select-none">
                  戦果
                </div>
                <h2
                  className={`text-8xl font-serif font-bold mb-6 tracking-wide uppercase ${battleScene.isVictory ? "text-gold" : "text-stone-700"}`}
                >
                  {battleScene.resultText}
                </h2>
                <p className="text-lg text-stone-500 mb-10 italic font-serif">
                  「
                  {Math.random() > 0.5
                    ? "これにて、この地の命運は決した。"
                    : "兵の鬨の声、大地に響き渡れり。"}
                  」
                </p>

                {/* 戦利品エリア */}
                {battleScene.isVictory && battleScene.loot && battleScene.loot.length > 0 && (
                  <div className="mb-10 max-w-xl mx-auto bg-black/40 border border-gold/15 p-5 rounded-sm text-left">
                    <h3 className="text-xs font-bold text-gold uppercase tracking-widest mb-4 flex items-center justify-center gap-2 font-serif border-b border-white/5 pb-2">
                      <Sparkles className="w-3.5 h-3.5 text-gold animate-pulse" />
                      獲得した戦利品 (Battle Spoils)
                    </h3>
                    <div className="grid grid-cols-1 sm:grid-cols-2 gap-3 font-sans">
                      {battleScene.loot.map((item: any, i: number) => {
                        let itemIcon = <Package className="w-4 h-4 text-stone-400" />;
                        let rarityBorder = "border-white/5 bg-white/5";
                        let nameColor = "text-stone-300";

                        if (item.type === "copper") {
                          itemIcon = <Coins className="w-4 h-4 text-orange-400" />;
                          rarityBorder = "border-orange-500/10 bg-orange-950/20";
                          nameColor = "text-orange-400 font-bold";
                        } else if (item.type === "gold") {
                          itemIcon = <Sparkles className="w-4 h-4 text-amber-400 animate-pulse" />;
                          rarityBorder = "border-amber-500/20 bg-amber-950/20";
                          nameColor = "text-gold font-bold";
                        } else if (item.type === "item") {
                          itemIcon = <ScrollText className="w-4 h-4 text-emerald-400" />;
                          rarityBorder = "border-emerald-500/10 bg-emerald-950/25";
                          nameColor = "text-emerald-400 font-bold";
                        } else if (item.type === "weapon") {
                          itemIcon = <Sword className="w-4 h-4 text-blue-400" />;
                          if (item.rarity === 5) {
                            rarityBorder = "border-amber-500/30 bg-amber-950/30 shadow-[inset_0_0_12px_rgba(245,158,11,0.1)]";
                            nameColor = "text-amber-400 font-bold flex items-center gap-1";
                          } else if (item.rarity === 4) {
                            rarityBorder = "border-purple-500/30 bg-purple-950/30 shadow-[inset_0_0_12px_rgba(168,85,247,0.1)]";
                            nameColor = "text-purple-400 font-bold";
                          } else {
                            rarityBorder = "border-blue-500/20 bg-blue-950/30";
                            nameColor = "text-cyan-400 font-bold";
                          }
                        }

                        return (
                          <div
                            key={i}
                            className={`flex items-center gap-3 p-2.5 border ${rarityBorder} rounded transition-all hover:scale-[1.02]`}
                          >
                            <div className="p-1.5 bg-black/40 rounded border border-white/5 shrink-0 flex items-center justify-center">
                              {itemIcon}
                            </div>
                            <div className="flex flex-col text-left">
                              <span className={`text-[12px] ${nameColor}`}>
                                {item.name}
                                {item.type === "weapon" && item.rarity === 5 && (
                                  <span className="text-[9px] bg-amber-950 text-gold px-1 py-0.5 rounded ml-1 uppercase font-serif leading-none">極</span>
                                )}
                              </span>
                              <span className="text-[10px] text-stone-500 font-mono">
                                {item.type === "copper" || item.type === "gold"
                                  ? `${item.count?.toLocaleString() || 0} 両`
                                  : `数量: ${item.count || 1}`}
                              </span>
                            </div>
                          </div>
                        );
                      })}
                    </div>
                  </div>
                )}

                <button
                  onClick={() => {
                    const wasConquestVictory =
                      battleScene.isVictory && !battleScene.isDefense;
                    const defeatedFaction = battleScene.land.owner;
                    setBattleScene(null);
                    setBattlingLandId(null);
                    if (wasConquestVictory) {
                      // 滅亡判定
                      let checkExtinguished = false;
                      if (defeatedFaction && defeatedFaction !== "群") {
                        const factionRemainingCount = lands.filter(
                          (l) => l.owner === defeatedFaction && !occupiedLands.includes(l.id)
                        ).length;
                        if (factionRemainingCount === 0) {
                          checkExtinguished = true;
                        }
                      }

                      const isClear = occupiedLands.length === lands.length;

                      if (checkExtinguished && !isClear) {
                        const defeatedLand = battleScene.land;
                        // 1. その土地に直接隣接する接続を取得
                        const neighboringLandIds = ROUTE_CONNECTIONS.filter(([a, b]) => {
                          return a === defeatedLand.id || b === defeatedLand.id;
                        }).map(([a, b]) => (a === defeatedLand.id ? b : a));

                        const neighboringLands = lands.filter((l) => neighboringLandIds.includes(l.id));

                        // 2. 隣接大名のうち、プレイヤー、群、および同盟以外の敵対大名をリストアップ
                        const threatLands = neighboringLands.filter((l) => {
                          return l.owner && 
                                 l.owner !== "群" && 
                                 l.owner !== defeatedFaction && 
                                 !occupiedLands.includes(l.id) && 
                                 diplomacyRelations[l.owner] !== "alliance";
                        });

                        let attackerFaction = "";
                        let triggerTargetLand = defeatedLand;

                        if (threatLands.length > 0) {
                          const randomThreat = threatLands[Math.floor(Math.random() * threatLands.length)];
                          attackerFaction = randomThreat.owner;
                        } else {
                          const otherHostileLands = lands.filter((l) => {
                            return l.owner && 
                                   l.owner !== "群" && 
                                   l.owner !== defeatedFaction && 
                                   !occupiedLands.includes(l.id) && 
                                   diplomacyRelations[l.owner] !== "alliance";
                          });
                          if (otherHostileLands.length > 0) {
                            const randomOther = otherHostileLands[Math.floor(Math.random() * otherHostileLands.length)];
                            attackerFaction = randomOther.owner;

                            const adjacentUserLands: typeof lands = [];
                            lands.filter(nl => nl.owner === attackerFaction).forEach(el => {
                              ROUTE_CONNECTIONS.forEach(([a, b]) => {
                                let userNeighborId = -1;
                                if (a === el.id) userNeighborId = b;
                                else if (b === el.id) userNeighborId = a;
                                if (userNeighborId !== -1 && occupiedLands.includes(userNeighborId)) {
                                  const pl = lands.find(l => l.id === userNeighborId);
                                  if (pl && !adjacentUserLands.some(exist => exist.id === pl.id)) {
                                    adjacentUserLands.push(pl);
                                  }
                                }
                              });
                            });
                            if (adjacentUserLands.length > 0) {
                              triggerTargetLand = adjacentUserLands[Math.floor(Math.random() * adjacentUserLands.length)];
                            }
                          }
                        }

                        addLog(
                          `【大名家滅亡】${defeatedFaction}家は、最後の拠点【${defeatedLand.name.split(" ")[0]}】を失い、ここに滅亡いたしました！`,
                          "success"
                        );

                        if (attackerFaction) {
                          const basePower = triggerTargetLand.power || 1000;
                          const enemyPower = Math.floor(basePower * (1.3 + Math.random() * 0.4));

                          setTimeout(() => {
                            setInvasionEvent({
                              land: triggerTargetLand,
                              enemyFaction: attackerFaction,
                              enemyPower: Math.max(enemyPower, 600),
                            });
                            addLog(
                              `【急告・連鎖侵攻】${defeatedFaction}家の滅亡を見て、勢力急拡大に恐怖した${attackerFaction}家が、我らが領国【${triggerTargetLand.name.split(" ")[0]}】へ火事場泥棒のごとく一斉侵攻を開始しました！`,
                              "error"
                            );
                          }, 1500);
                        } else {
                          addLog(
                            `【天下震撼】我が軍が${defeatedFaction}家を滅亡させたことで、近国の大名らは我らの圧倒的な武威に恐れおののいています。`,
                            "success"
                          );
                        }
                      } else {
                        if (Math.random() * 100 < 20) {
                          setTimeout(() => {
                            triggerInvasionCheck();
                          }, 1200);
                        }
                      }
                    }
                  }}
                  className="px-16 py-4 bg-gold text-black font-bold text-sm rounded-sm transition-all uppercase tracking-[0.3em] hover:bg-[#D4A159] hover:shadow-[0_0_30px_rgba(197,160,89,0.3)] cursor-pointer"
                >
                  本陣へ帰還
                </button>
              </motion.div>
            )}
          </motion.div>
        )}
      </AnimatePresence>

      {/* Gacha Result Layer */}
      <AnimatePresence>
        {gachaResult && (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            className="absolute inset-0 z-[60] bg-black/95 flex flex-col items-center justify-center p-12 overflow-y-auto"
          >
            <h2 className="text-4xl font-black text-amber-500 mb-12 tracking-widest">
              登用結果
            </h2>
            <div className="flex flex-wrap justify-center gap-8 max-w-6xl">
              {gachaResult.map((g, i) => (
                <motion.div
                  key={i}
                  initial={{ y: 50, opacity: 0 }}
                  animate={{ y: 0, opacity: 1 }}
                  transition={{ delay: i * 0.1 }}
                  className="bg-stone-900 border-2 border-stone-800 p-6 rounded-2xl flex flex-col items-center min-w-[200px] relative"
                >
                  <div className="mb-4">
                    <RarityStars count={g.rarity} />
                  </div>
                  <div className="absolute top-4 right-4 opacity-20">
                    <FactionKamon faction={g.faction} className="w-12 h-12" />
                  </div>
                  <h4
                    className={`text-2xl font-bold mb-2 ${g.rarity >= 4 ? "text-amber-400" : "text-white"}`}
                  >
                    {g.name}
                  </h4>
                  <span className="text-xs text-stone-500 mb-6">
                    {g.faction}
                  </span>
                  <div
                    className={`text-xs px-4 py-1 rounded-full font-bold uppercase tracking-widest ${g.troop.color}`}
                  >
                    {g.troop.name}隊
                  </div>
                </motion.div>
              ))}
            </div>
            <button
              onClick={() => setGachaResult(null)}
              className="mt-16 px-12 py-4 bg-amber-900/50 hover:bg-amber-800 border-2 border-amber-600 text-amber-100 font-bold rounded-full transition-all"
            >
              確認
            </button>
          </motion.div>
        )}
      </AnimatePresence>

      {/* Enemy Invasion Layer */}
      <AnimatePresence>
        {invasionEvent &&
          (() => {
            const bribeCost = Math.max(invasionEvent.land.level * 150, 400);
            const hasBribeFunds = gold >= bribeCost;
            return (
              <motion.div
                initial={{ opacity: 0 }}
                animate={{ opacity: 1 }}
                exit={{ opacity: 0 }}
                className="absolute inset-0 z-[70] bg-black/90 flex items-center justify-center p-6 backdrop-blur-md"
              >
                <div
                  id="invasion_alert_modal"
                  className="bg-[#1c1917] border-2 border-red-900/60 p-8 md:p-12 rounded-sm shadow-[0_0_50px_rgba(239,68,68,0.3)] max-w-2xl w-full flex flex-col relative overflow-hidden"
                >
                  {/* Visual alert highlights */}
                  <div className="absolute top-0 inset-x-0 h-1.5 bg-gradient-to-r from-red-600 via-amber-500 to-red-600 animate-pulse"></div>
                  <div className="absolute -top-12 -right-12 opacity-5 text-red-500 text-9xl font-bold font-serif select-none pointer-events-none">
                    襲
                  </div>

                  <div className="flex flex-col items-center text-center mb-8">
                    <div className="w-16 h-16 rounded-full bg-red-950/40 border border-red-500 flex items-center justify-center mb-4 shadow-[0_0_15px_rgba(239,68,68,0.4)] animate-pulse">
                      <AlertTriangle className="w-8 h-8 text-red-500" />
                    </div>
                    <h2 className="text-3xl font-serif font-bold text-red-500 tracking-widest uppercase">
                      戦国急報：敵襲来！
                    </h2>
                    <p className="text-stone-500 font-sans text-xs mt-1 block uppercase tracking-[0.2em]">
                      Sengoku Emergency Defense Command
                    </p>
                  </div>

                  <div className="bg-black/30 border border-white/5 rounded-sm p-6 mb-8 font-sans">
                    <div className="flex items-center justify-between border-b border-white/5 pb-4 mb-4">
                      <div className="flex items-center gap-3">
                        <span className="text-stone-500 text-xs text-stone-500">
                          侵撃大名：
                        </span>
                        <span className="text-sm font-serif font-bold text-stone-200">
                          {invasionEvent.enemyFaction}家
                        </span>
                      </div>
                      <div className="flex items-center gap-3">
                        <span className="text-stone-500 text-xs text-stone-500">
                          目標土地：
                        </span>
                        <span className="text-sm font-serif font-bold text-red-400">
                          {invasionEvent.land.name.split(" ")[0]}
                        </span>
                      </div>
                    </div>

                    <p className="text-xs text-stone-300 leading-relaxed mb-6">
                      報告！我が領国【
                      <span className="text-red-400 font-bold">
                        {invasionEvent.land.name.split(" ")[0]}
                      </span>
                      】に向けて、
                      <span className="text-amber-500 font-bold">
                        {invasionEvent.enemyFaction}軍
                      </span>
                      が兵力{" "}
                      <span className="font-mono text-red-400 font-bold">
                        {invasionEvent.enemyPower.toLocaleString()}
                      </span>{" "}
                      を率いて侵略の駒を進めております！
                      <br />
                      このまま防兵を派遣して迎え撃つか、あるいは領土を差し出して一時撤退するか、もしくは小判を支払って和睦を締結せねば軍を留められませぬ！
                    </p>

                    <div className="border-t border-dashed border-stone-800 pt-4">
                      <div className="text-[10px] text-stone-500 font-mono tracking-widest uppercase mb-3 block">
                        ● 迎撃隊の編制を選択：
                      </div>
                      <div className="grid grid-cols-1 md:grid-cols-3 gap-3">
                        {[0, 1, 2].map((idx) => {
                          const power = calculateDeckPower(idx);
                          const isDeckEmpty =
                            decks[idx].filter((id) => id).length === 0;
                          const isSelected = selectedDeckForBattle === idx;
                          return (
                            <button
                              key={idx}
                              disabled={isDeckEmpty}
                              onClick={() => setSelectedDeckForBattle(idx)}
                              className={`p-3 border rounded-sm flex flex-col items-center justify-center transition-all ${
                                isDeckEmpty
                                  ? "bg-stone-900 border-white/5 text-stone-600 cursor-not-allowed text-center"
                                  : isSelected
                                    ? "bg-red-950/40 border-red-500 text-white shadow-[0_0_15px_rgba(220,38,38,0.2)]"
                                    : "bg-stone-900 border-stone-800 text-stone-400 hover:border-stone-500 hover:text-stone-200"
                              }`}
                            >
                              <span className="text-[9px] block uppercase font-bold tracking-widest mb-1">
                                第 {idx + 1} 部隊
                              </span>
                              <span className="text-[11px] font-mono font-bold">
                                {isDeckEmpty
                                  ? "空部隊"
                                  : `兵力: ${power.toLocaleString()}`}
                              </span>
                            </button>
                          );
                        })}
                      </div>
                    </div>
                  </div>

                  <div className="flex flex-col sm:flex-row gap-4 justify-between mt-auto">
                    {/* 退却 */}
                    <button
                      onClick={() => {
                        setOccupiedLands((prev) =>
                          prev.filter((id) => id !== invasionEvent.land.id),
                        );
                        setInvasionEvent(null);
                        addLog(
                          `【防衛放棄】我らは領土【${invasionEvent.land.name.split(" ")[0]}】の防衛を諦め、一時退避いたしました。領地は ${invasionEvent.enemyFaction}軍 に制圧されました。`,
                          "error",
                        );
                      }}
                      className="flex-1 px-4 py-3 bg-stone-900 border border-stone-700 text-stone-400 hover:text-stone-200 font-serif font-bold text-xs tracking-widest hover:border-stone-500 transition-all select-none"
                    >
                      一時退去する（領地放棄）
                    </button>

                    {/* 和睦 */}
                    <button
                      disabled={!hasBribeFunds}
                      onClick={() => {
                        setGold((prev) => prev - bribeCost);
                        setInvasionEvent(null);
                        addLog(
                          `【金員和睦】和睦の約定を交わし、小判 ${bribeCost.toLocaleString()} 枚を贈って ${invasionEvent.enemyFaction}軍 を撤退させました。`,
                          "success",
                        );
                      }}
                      className={`flex-1 px-4 py-3 border font-serif font-bold text-xs tracking-widest transition-all select-none ${
                        hasBribeFunds
                          ? "bg-amber-950 border-amber-500 text-amber-400 hover:bg-amber-500 hover:text-black hover:border-amber-500"
                          : "bg-stone-900 border-white/5 text-stone-500 cursor-not-allowed"
                      }`}
                    >
                      和睦を仲介（小判 -{bribeCost.toLocaleString()}枚）
                    </button>

                    {/* 迎撃 */}
                    <button
                      onClick={() => {
                        const deckIdx = selectedDeckForBattle;
                        // Close invasion overlay, triggers direct battle screen
                        startBattle(
                          invasionEvent.land,
                          deckIdx,
                          true,
                          invasionEvent.enemyPower,
                          `${invasionEvent.enemyFaction}侵攻軍`,
                        );
                      }}
                      className="flex-1 px-6 py-3 bg-red-800 border border-red-700 text-white hover:bg-red-700 font-serif font-bold text-xs lg:text-sm tracking-widest shadow-[0_0_15px_rgba(220,38,38,0.3)] hover:shadow-[0_0_25px_rgba(220,38,38,0.5)] transition-all select-none"
                    >
                      出撃し 迎撃せよ！
                    </button>
                  </div>
                </div>
              </motion.div>
            );
          })()}
      </AnimatePresence>

      <style>{`
        .origin-top-left { transform-origin: top left; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { border-radius: 10px; background-color: #444; }
      `}</style>
    </div>
  );

  async function handleConsultAdvisor() {
    if (!advisorQuery.trim() || isConsulting) return;
    setIsConsulting(true);
    setAdvisorResponse("");

    const deckInfo = decks
      .map((deck, idx) => {
        const genStr = deck
          .filter((id) => id)
          .map((id) => {
            const g = ownedGenerals.find((og) => og.uniqueId === id);
            return g ? `${g.name}(Lv${g.level}/${g.troop.name})` : "";
          })
          .join(", ");
        return `第${idx + 1}部隊: ${genStr || "空"}`;
      })
      .join(" | ");

    const promptText = `我が軍の現状：【編制】${deckInfo} 【銅銭】${copper} 【小判】${gold} 【領地数】${occupiedLands.length}。相談内容：${advisorQuery}`;
    const result = await callAI("advisor", promptText);
    setAdvisorResponse(result);
    setIsConsulting(false);
  }
}
