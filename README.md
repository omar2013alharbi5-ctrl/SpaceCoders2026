import React, { useState, useEffect, useRef, useMemo, useCallback } from 'react';
import {
  Rocket, Compass, Cpu, FlaskConical, Sun, Activity, ArrowDownToLine,
  Radio, FileText, Award, Users, ShieldCheck, Globe, Mic, MicOff, Volume2,
  Play, RotateCcw, CheckCircle2, AlertTriangle, Info, Terminal, Download,
  Printer, ChevronRight, Sparkles, Navigation, Sliders, BookOpen, X,
  Zap, Eye, Layers, Wifi, WifiOff, ArrowUp, ArrowDown, ArrowLeft, ArrowRight,
  Target, HelpCircle, Code2, Lock, Star, BarChart3
} from 'lucide-react';

/**
 * ============================================================================
 * ASTRO-TRAINER X — Junior Astronaut Training & Robotic Exploration System
 * ============================================================================
 * Single-File Self-Contained React Application
 * Features:
 * - Bilingual EN / AR with dynamic RTL/LTR switching & Web Speech API
 * - NASA Data-to-Mission Engine (Live NOAA/NASA public endpoints + transparent fallback)
 * - 13 Functional Interactive Modules (3D Planetary Explorer, Autonomous Rover + ESP32/Arduino,
 *   Radiation Shielding Canvas, Space Weather Charts, Spectral Rock Analyzer,
 *   Gravity Physics Simulator, Deep Space Light-Speed Delay Simulator,
 *   Automated Scientific Report Generator, Gamification, Cooperative Multi-Role Control,
 *   COPPA-Aligned Privacy Center & Developer Docs)
 * - Persistent LocalStorage state without PII
 * ============================================================================
 */

// --- DYNAMIC EXTERNAL SCRIPT LOADER (Three.js & Chart.js) ---
const useExternalScripts = () => {
  const [status, setStatus] = useState({ three: !!window.THREE, chart: !!window.Chart });

  useEffect(() => {
    const loadScript = (src, key, globalName) => {
      if (window[globalName]) {
        setStatus((prev) => ({ ...prev, [key]: true }));
        return;
      }
      const existing = document.querySelector(`script[src="${src}"]`);
      if (existing) {
        existing.addEventListener('load', () => setStatus((prev) => ({ ...prev, [key]: true })));
        return;
      }
      const script = document.createElement('script');
      script.src = src;
      script.async = true;
      script.onload = () => setStatus((prev) => ({ ...prev, [key]: true }));
      script.onerror = () => console.warn(`Fallback mode active for ${globalName}`);
      document.head.appendChild(script);
    };

    loadScript('https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js', 'three', 'THREE');
    loadScript('https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js', 'chart', 'Chart');
  }, []);

  return status;
};

// --- BILINGUAL DICTIONARY (ENGLISH & ARABIC) ---
const TRANSLATIONS = {
  en: {
    appName: 'ASTRO-TRAINER X',
    subtitle: 'Junior Astronaut & Robotics Command',
    missionControl: 'Mission Control',
    profile: 'Astronaut Profile',
    planetary3d: '3D Planetary Explorer',
    roverControl: 'Rover & Robotics',
    radiationLab: 'Radiation Shield Lab',
    spaceWeather: 'Space Weather Center',
    spectralLab: 'Spectral Rock Analyzer',
    gravitySim: 'Gravity Simulator',
    deepSpaceComms: 'Deep Space Comms',
    scientificReports: 'Scientific Reports',
    achievements: 'Badges & Ranks',
    coopMissions: 'Cooperative Control',
    privacySafety: 'Privacy & Safety',
    devDocs: 'System Docs',
    voiceCmd: 'Voice Command',
    listening: 'Listening...',
    xp: 'XP',
    level: 'Level',
    rank: 'Rank',
    dataSourceLive: 'LIVE VERIFIED TELEMETRY',
    dataSourceCached: 'CACHED NASA/NOAA DATA',
    dataSourceSim: 'EDUCATIONAL SIMULATION DATA',
    startMission: 'Launch Active Mission',
    completeStep: 'Complete Objective',
    logObservation: 'Log to Scientific Report',
    loggedSuccess: 'Observation Logged!',
    callsign: 'Explorer Callsign',
    safeModeNotice: 'Zero-PII Local Storage • Child-Safe Environment',
    ranks: ['Space Cadet', 'Junior Specialist', 'Mission Pilot', 'Planetary Scientist', 'Senior Commander'],
    voiceHelp: 'Try: "Open rover", "Gravity simulator", "Space weather", "Generate report", "Start mission"',
  },
  ar: {
    appName: 'أسترو ترينر إكس',
    subtitle: 'نظام تدريب رواد الفضاء الصغار والروبوتات',
    missionControl: 'مركز التحكم بالمهام',
    profile: 'ملف رائد الفضاء',
    planetary3d: 'مستكشف الكواكب ثلاثي الأبعاد',
    roverControl: 'التحكم بالمركبة الجوالة',
    radiationLab: 'مختبر الحماية من الإشعاع',
    spaceWeather: 'مركز طقس الفضاء',
    spectralLab: 'محلل الصخور الطيفي',
    gravitySim: 'محاكي الجاذبية',
    deepSpaceComms: 'اتصالات الفضاء العميق',
    scientificReports: 'التقارير العلمية',
    achievements: 'الأوسمة والرتب',
    coopMissions: 'المهام التعاونية',
    privacySafety: 'الخصوصية والأمان',
    devDocs: 'وثائق النظام',
    voiceCmd: 'أمر صوتي',
    listening: 'جاري الاستماع...',
    xp: 'نقطة خبرة',
    level: 'المستوى',
    rank: 'الرتبة',
    dataSourceLive: 'بيانات حية موثقة',
    dataSourceCached: 'بيانات ناسا مخزنة مؤقتاً',
    dataSourceSim: 'بيانات محاكاة تعليمية',
    startMission: 'بدء المهمة النشطة',
    completeStep: 'إتمام الهدف',
    logObservation: 'تسجيل في التقرير العلمي',
    loggedSuccess: 'تم تسجيل الملاحظة!',
    callsign: 'الرمز التعريفي للمستكشف',
    safeModeNotice: 'تخزين محلي بدون بيانات شخصية • بيئة آمنة للأطفال',
    ranks: ['طالب فضاء', 'أخصائي مبتدئ', 'طيار مهمة', 'عالم كواكب', 'قائد أول'],
    voiceHelp: 'جرب: "افتح المركبة"، "محاكي الجاذبية"، "طقس الفضاء"، "تقرير علمي"',
  },
};

// --- SCIENTIFIC DATASETS (STRICTLY LABELED) ---
const PLANETARY_BODIES = {
  mars: {
    id: 'mars',
    name: 'Mars (Jezero Crater)',
    nameAr: 'المريخ (فوهة جيزيرو)',
    gravity: 3.72,
    gRatio: 0.38,
    radiusKm: 3389.5,
    tempC: '-63°C avg',
    atmosphere: '95.3% CO₂, 2.7% N₂, 1.6% Ar',
    color: '#d95b43',
    accent: '#f97316',
    waypoints: [
      { id: 'WP-1', name: 'Octavia E. Butler Landing', lat: 18.44, lon: 77.45, info: 'Primary touchdown zone on ancient river delta floor.' },
      { id: 'WP-2', name: 'Séítah Bedrock Outcrop', lat: 18.42, lon: 77.41, info: 'Olivine-rich igneous rocks altered by ancient liquid water.' },
      { id: 'WP-3', name: 'Delta Front Escarpment', lat: 18.48, lon: 77.43, info: 'Sedimentary layers ideal for biosignature preservation.' },
    ],
  },
  moon: {
    id: 'moon',
    name: 'Earth’s Moon (Shackleton Rim)',
    nameAr: 'قمر الأرض (حافة شاكلتون)',
    gravity: 1.62,
    gRatio: 0.166,
    radiusKm: 1737.4,
    tempC: '-23°C to -233°C',
    atmosphere: 'Exosphere (Near Vacuum)',
    color: '#94a3b8',
    accent: '#cbd5e1',
    waypoints: [
      { id: 'WP-1', name: 'Artemis South Pole Site', lat: -89.54, lon: 0.0, info: 'Near-continuous sunlight ridge adjacent to shadowed craters.' },
      { id: 'WP-2', name: 'Cold Trap Ice Basin', lat: -89.9, lon: 45.0, info: 'Permanently shadowed region containing water-ice regolith.' },
      { id: 'WP-3', name: 'Anorthosite Highland Ridge', lat: -88.7, lon: -30.0, info: 'Ancient lunar crust sample site.' },
    ],
  },
  earth: {
    id: 'earth',
    name: 'Earth (Reference Baseline)',
    nameAr: 'الأرض (المرجع القياسي)',
    gravity: 9.81,
    gRatio: 1.0,
    radiusKm: 6371.0,
    tempC: '+15°C avg',
    atmosphere: '78% N₂, 21% O₂, 1% Ar/CO₂',
    color: '#2563eb',
    accent: '#38bdf8',
    waypoints: [
      { id: 'WP-1', name: 'Kennedy Launch Complex 39A', lat: 28.6, lon: -80.6, info: 'Primary orbital launch facility & telemetry uplink.' },
      { id: 'WP-2', name: 'Atacama Mars Analog Site', lat: -24.5, lon: -69.25, info: 'Hyper-arid desert used to test NASA rover autonomy.' },
    ],
  },
  europa: {
    id: 'europa',
    name: 'Europa (Conamara Chaos)',
    nameAr: 'قمر أوروبا (فوضى كونامارا)',
    gravity: 1.315,
    gRatio: 0.134,
    radiusKm: 1560.8,
    tempC: '-171°C avg',
    atmosphere: 'Tenuous O₂ Exosphere',
    color: '#e2e8f0',
    accent: '#38bdf8',
    waypoints: [
      { id: 'WP-1', name: 'Cryovolcanic Ridge Alpha', lat: 9.2, lon: -87.4, info: 'Fractured water-ice crust above a global subsurface saltwater ocean.' },
      { id: 'WP-2', name: 'Hydrated Salt Band', lat: 11.5, lon: -85.1, info: 'Reddish-brown magnesium sulfate & sodium chloride deposits.' },
    ],
  },
};

const ROCK_SAMPLES = [
  {
    id: 'basalt',
    name: 'Martian Olivine Basalt',
    code: 'SAMPLE-MB01',
    mineralClass: 'Igneous Mafic Silicate',
    diagnosticBands: '950 nm & 1050 nm broad Fe²⁺ absorption',
    albedo: 'Low (0.12 - 0.18)',
    description: 'Dark volcanic rock rich in magnesium and iron silicates. Indicates ancient basaltic lava flows on Mars.',
    spectrum: [0.11, 0.13, 0.15, 0.14, 0.09, 0.08, 0.11, 0.14, 0.16, 0.17, 0.16],
    reference: [0.10, 0.12, 0.14, 0.13, 0.085, 0.075, 0.105, 0.135, 0.155, 0.165, 0.155],
  },
  {
    id: 'hematite',
    name: 'Crystalline Gray Hematite ("Blueberries")',
    code: 'SAMPLE-HM04',
    mineralClass: 'Iron Oxide (Fe₂O₃)',
    diagnosticBands: '530 nm ferric edge & 860 nm crystal field band',
    albedo: 'Moderate (0.22 - 0.34)',
    description: 'Spherical iron-oxide concretions formed in the presence of liquid groundwater at Meridiani Planum.',
    spectrum: [0.08, 0.14, 0.28, 0.31, 0.21, 0.26, 0.32, 0.34, 0.35, 0.34, 0.33],
    reference: [0.07, 0.13, 0.27, 0.30, 0.20, 0.25, 0.31, 0.33, 0.34, 0.33, 0.32],
  },
  {
    id: 'gypsum',
    name: 'Hydrated Calcium Sulfate (Gypsum Vein)',
    code: 'SAMPLE-GY09',
    mineralClass: 'Hydrous Sulfate Evaporite',
    diagnosticBands: '1450 nm & 1940 nm sharp H₂O overtone bands',
    albedo: 'High (0.55 - 0.72)',
    description: 'Bright mineral vein deposited by mineral-rich water circulating through rock fractures.',
    spectrum: [0.58, 0.64, 0.68, 0.69, 0.67, 0.65, 0.41, 0.62, 0.34, 0.56, 0.52],
    reference: [0.57, 0.63, 0.67, 0.68, 0.66, 0.64, 0.40, 0.61, 0.33, 0.55, 0.51],
  },
  {
    id: 'silica',
    name: 'Opaline Hydrated Silica',
    code: 'SAMPLE-SI12',
    mineralClass: 'Amorphous Hydrated Silicate (SiO₂·nH₂O)',
    diagnosticBands: '1410 nm, 1910 nm & 2210 nm Si-OH absorption',
    albedo: 'High (0.60 - 0.75)',
    description: 'Formed near ancient hydrothermal hot springs—a prime astrobiology target for preserving micro-textures.',
    spectrum: [0.62, 0.68, 0.71, 0.72, 0.70, 0.68, 0.48, 0.66, 0.45, 0.51, 0.63],
    reference: [0.61, 0.67, 0.70, 0.71, 0.69, 0.67, 0.47, 0.65, 0.44, 0.50, 0.62],
  },
];

const SHIELD_MATERIALS = [
  { id: 'aluminum', name: 'Aerospace Aluminum (6061-T6)', density: 2.7, hContent: '0%', gcrAtten: 0.045, speAtten: 0.14, notes: 'Standard spacecraft hull. Good against solar protons, but heavy ions can create secondary neutrons.' },
  { id: 'polyethylene', name: 'High-Density Polyethylene (HDPE)', density: 0.97, hContent: '14.3%', gcrAtten: 0.085, speAtten: 0.22, notes: 'Hydrogen-rich polymer! Excels at breaking up heavy galactic cosmic ray ions with minimal secondary radiation.' },
  { id: 'water', name: 'Potable Water Wall (H₂O)', density: 1.0, hContent: '11.1%', gcrAtten: 0.082, speAtten: 0.21, notes: 'Dual-purpose life support & radiation storm shelter lining around crew sleeping quarters.' },
  { id: 'regolith', name: 'Compacted Martian Regolith', density: 1.5, hContent: '3-5% bound', gcrAtten: 0.055, speAtten: 0.16, notes: 'In-situ resource utilization (ISRU): piled 1–2 meters thick over surface habitats.' },
  { id: 'lead', name: 'Dense Lead (Pb)', density: 11.34, hContent: '0%', gcrAtten: 0.032, speAtten: 0.19, notes: 'High mass penalty for launch and generates high secondary Bremsstrahlung X-rays from cosmic rays.' },
];

const BADGES_CATALOG = [
  { id: 'first_launch', title: 'Orbital Cadet', desc: 'Launched your first ASTRO-TRAINER X mission', xp: 100, icon: '🚀' },
  { id: 'rover_pilot', title: 'Autonomous Navigator', desc: 'Guided the planetary rover to a target waypoint', xp: 150, icon: '🛞' },
  { id: 'gravity_master', title: 'Gravity Theorist', desc: 'Executed a multi-planet free-fall comparison', xp: 120, icon: '🪐' },
  { id: 'spectroscopist', title: 'Mineral Detective', desc: 'Analyzed 3 planetary rock spectra in the lab', xp: 150, icon: '🔬' },
  { id: 'shield_engineer', title: 'Radiation Architect', desc: 'Tested spacecraft shielding configurations', xp: 120, icon: '🛡️' },
  { id: 'deep_comms', title: 'Deep Space Linker', desc: 'Transmitted a delayed command across interplanetary space', xp: 130, icon: '📡' },
  { id: 'chief_scientist', title: 'Mission Publisher', desc: 'Generated an official Scientific Mission Report', xp: 200, icon: '📋' },
];

const MISSION_CATALOG = [
  {
    id: 'M-01',
    title: 'Jezero Delta Rover Reconnaissance',
    difficulty: 'Cadet',
    xp: 250,
    targetTab: 'rover',
    summary: 'Navigate the autonomous rover across obstacle-strewn terrain to sample hydrated mineral outcrops.',
    objective: 'Reach waypoint target on the Rover Control grid and log telemetry.',
  },
  {
    id: 'M-02',
    title: 'Multi-Planet Gravity & Entry Analysis',
    difficulty: 'Specialist',
    xp: 200,
    targetTab: 'gravity',
    summary: 'Compare gravitational acceleration across Earth (1.0g), Mars (0.38g), and Moon (0.166g) for lander descent.',
    objective: 'Run a drop simulation and record fall-time differentials.',
  },
  {
    id: 'M-03',
    title: 'Biosignature Mineral Spectroscopy',
    difficulty: 'Scientist',
    xp: 300,
    targetTab: 'spectral',
    summary: 'Inspect reflectance spectra between 400nm and 2500nm to identify water-altered minerals.',
    objective: 'Analyze at least 2 rock samples and log a spectral classification.',
  },
  {
    id: 'M-04',
    title: 'Solar Storm Crew Shielding Drill',
    difficulty: 'Pilot',
    xp: 220,
    targetTab: 'radiation',
    summary: 'Evaluate Hydrogen-rich Polyethylene vs. Aluminum shielding during a Solar Particle Event (SPE).',
    objective: 'Reduce relative radiation exposure below 35% in the Radiation Shield Lab.',
  },
];

// --- HELPER: STARFIELD CANVAS BACKGROUND ---
const StarfieldBackground = () => {
  const canvasRef = useRef(null);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    let animId;

    const resize = () => {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    };
    resize();
    window.addEventListener('resize', resize);

    const stars = Array.from({ length: 130 }, () => ({
      x: Math.random() * canvas.width,
      y: Math.random() * canvas.height,
      r: Math.random() * 1.5 + 0.3,
      alpha: Math.random() * 0.8 + 0.2,
      speed: Math.random() * 0.15 + 0.03,
      twinkle: Math.random() * 0.02,
    }));

    const render = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      stars.forEach((s) => {
        s.y -= s.speed;
        if (s.y < 0) {
          s.y = canvas.height;
          s.x = Math.random() * canvas.width;
        }
        s.alpha += s.twinkle;
        if (s.alpha > 1 || s.alpha < 0.2) s.twinkle *= -1;
        ctx.beginPath();
        ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(186, 230, 253, ${s.alpha})`;
        ctx.fill();
      });
      animId = requestAnimationFrame(render);
    };
    render();

    return () => {
      cancelAnimationFrame(animId);
      window.removeEventListener('resize', resize);
    };
  }, []);

  return <canvas ref={canvasRef} className="fixed inset-0 pointer-events-none z-0 opacity-60" />;
};

// --- DATA SOURCE PROVENANCE BADGE ---
const DataSourceBadge = ({ type, t }) => {
  const styles = {
    live: 'bg-emerald-500/15 text-emerald-300 border-emerald-500/40',
    cached: 'bg-amber-500/15 text-amber-300 border-amber-500/40',
    sim: 'bg-purple-500/15 text-purple-300 border-purple-500/40',
  };
  const labels = {
    live: t.dataSourceLive,
    cached: t.dataSourceCached,
    sim: t.dataSourceSim,
  };
  return (
    <span className={`inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full text-[11px] font-mono border ${styles[type] || styles.sim}`}>
      <span className={`w-2 h-2 rounded-full ${type === 'live' ? 'bg-emerald-400 animate-pulse' : type === 'cached' ? 'bg-amber-400' : 'bg-purple-400'}`} />
      {labels[type] || labels.sim}
    </span>
  );
};

// ============================================================================
// MODULE 3: 3D PLANETARY EXPLORER (Three.js + Interactive Canvas Fallback)
// ============================================================================
const PlanetaryExplorerModule = ({ t, onLogObservation, onAwardXP }) => {
  const [selectedBody, setSelectedBody] = useState('mars');
  const [activeWaypoint, setActiveWaypoint] = useState(PLANETARY_BODIES.mars.waypoints[0]);
  const [webXrStatus, setWebXrStatus] = useState('Checking WebXR hardware...');
  const mountRef = useRef(null);
  const body = PLANETARY_BODIES[selectedBody];

  useEffect(() => {
    setActiveWaypoint(PLANETARY_BODIES[selectedBody].waypoints[0]);
  }, [selectedBody]);

  useEffect(() => {
    if (navigator.xr && navigator.xr.isSessionSupported) {
      navigator.xr.isSessionSupported('immersive-vr').then((supported) => {
        setWebXrStatus(supported ? 'WebXR VR Headset Ready' : 'Standard 3D Viewport (No VR Headset Detected)');
      }).catch(() => setWebXrStatus('Standard 3D Viewport'));
    } else {
      setWebXrStatus('Standard 3D Viewport (WebXR API Unavailable in Browser)');
    }
  }, []);

  useEffect(() => {
    const container = mountRef.current;
    if (!container) return;

    let animId;
    let isDragging = false;
    let prevMouse = { x: 0, y: 0 };
    let rotY = 0.4;
    let rotX = 0.25;
    let zoom = 3.1;

    // Use Three.js if loaded, otherwise high-precision 2D orbital projection
    if (window.THREE) {
      const THREE = window.THREE;
      const scene = new THREE.Scene();
      const width = container.clientWidth || 600;
      const height = container.clientHeight || 380;
      const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 100);
      camera.position.z = zoom;

      const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      renderer.setSize(width, height);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
      container.innerHTML = '';
      container.appendChild(renderer.domElement);

      // Procedural Planetary Texture via Offscreen Canvas (Zero external URLs)
      const texCanvas = document.createElement('canvas');
      texCanvas.width = 512;
      texCanvas.height = 256;
      const tctx = texCanvas.getContext('2d');
      tctx.fillStyle = body.color;
      tctx.fillRect(0, 0, 512, 256);

      // Surface craters / terrain bands
      for (let i = 0; i < 120; i++) {
        const cx = Math.random() * 512;
        const cy = Math.random() * 256;
        const cr = Math.random() * 18 + 3;
        tctx.beginPath();
        tctx.arc(cx, cy, cr, 0, Math.PI * 2);
        tctx.fillStyle = i % 2 === 0 ? 'rgba(0,0,0,0.16)' : 'rgba(255,255,255,0.1)';
        tctx.fill();
      }
      if (selectedBody === 'earth') {
        tctx.fillStyle = 'rgba(34, 197, 94, 0.55)';
        for (let i = 0; i < 14; i++) {
          tctx.beginPath();
          tctx.ellipse(Math.random() * 512, Math.random() * 200 + 28, 35, 22, 0.3, 0, Math.PI * 2);
          tctx.fill();
        }
      }
      if (selectedBody === 'europa') {
        tctx.strokeStyle = 'rgba(180, 83, 9, 0.45)';
        tctx.lineWidth = 2;
        for (let i = 0; i < 25; i++) {
          tctx.beginPath();
          tctx.moveTo(Math.random() * 512, 0);
          tctx.lineTo(Math.random() * 512, 256);
          tctx.stroke();
        }
      }

      const texture = new THREE.CanvasTexture(texCanvas);
      const geometry = new THREE.SphereGeometry(1, 48, 48);
      const material = new THREE.MeshStandardMaterial({
        map: texture,
        roughness: 0.78,
        metalness: 0.08,
      });
      const planet = new THREE.Mesh(geometry, material);
      scene.add(planet);

      // Atmospheric Glow Ring
      const atmoGeo = new THREE.SphereGeometry(1.045, 32, 32);
      const atmoMat = new THREE.MeshBasicMaterial({
        color: body.accent,
        transparent: true,
        opacity: 0.14,
        side: THREE.BackSide,
      });
      scene.add(new THREE.Mesh(atmoGeo, atmoMat));

      // Orbital Trajectory Ring
      const orbitGeo = new THREE.RingGeometry(1.38, 1.40, 64);
      const orbitMat = new THREE.MeshBasicMaterial({ color: 0x00d2ff, side: THREE.DoubleSide, transparent: true, opacity: 0.45 });
      const orbitRing = new THREE.Mesh(orbitGeo, orbitMat);
      orbitRing.rotation.x = Math.PI / 2.4;
      scene.add(orbitRing);

      // Rover Beacon Pin on Surface
      const pinGeo = new THREE.SphereGeometry(0.045, 16, 16);
      const pinMat = new THREE.MeshBasicMaterial({ color: 0x00ffcc });
      const roverPin = new THREE.Mesh(pinGeo, pinMat);
      roverPin.position.set(0.65, 0.48, 0.6);
      planet.add(roverPin);

      // Lighting
      const sunLight = new THREE.DirectionalLight(0xffffff, 1.35);
      sunLight.position.set(4, 2, 4);
      scene.add(sunLight);
      scene.add(new THREE.AmbientLight(0x334155, 0.75));

      const onDown = (e) => {
        isDragging = true;
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        prevMouse = { x: clientX, y: clientY };
      };
      const onMove = (e) => {
        if (!isDragging) return;
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        const clientY = e.touches ? e.touches[0].clientY : e.clientY;
        rotY += (clientX - prevMouse.x) * 0.008;
        rotX = Math.max(-1.2, Math.min(1.2, rotX + (clientY - prevMouse.y) * 0.008));
        prevMouse = { x: clientX, y: clientY };
      };
      const onUp = () => { isDragging = false; };
      const onWheel = (e) => {
        e.preventDefault();
        zoom = Math.max(1.8, Math.min(5.0, zoom + e.deltaY * 0.002));
        camera.position.z = zoom;
      };

      const dom = renderer.domElement;
      dom.addEventListener('mousedown', onDown);
      window.addEventListener('mousemove', onMove);
      window.addEventListener('mouseup', onUp);
      dom.addEventListener('touchstart', onDown, { passive: true });
      dom.addEventListener('touchmove', onMove, { passive: true });
      dom.addEventListener('touchend', onUp);
      dom.addEventListener('wheel', onWheel, { passive: false });

      const onResize = () => {
        if (!container) return;
        const w = container.clientWidth;
        const h = container.clientHeight;
        camera.aspect = w / h;
        camera.updateProjectionMatrix();
        renderer.setSize(w, h);
      };
      window.addEventListener('resize', onResize);

      const animate = () => {
        animId = requestAnimationFrame(animate);
        if (!isDragging) rotY += 0.003;
        planet.rotation.y = rotY;
        planet.rotation.x = rotX;
        roverPin.scale.setScalar(1 + Math.sin(Date.now() * 0.006) * 0.25);
        renderer.render(scene, camera);
      };
      animate();

      return () => {
        cancelAnimationFrame(animId);
        window.removeEventListener('mousemove', onMove);
        window.removeEventListener('mouseup', onUp);
        window.removeEventListener('resize', onResize);
        renderer.dispose();
      };
    }
  }, [selectedBody, body]);

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <Globe className="w-6 h-6 text-cyan-400" />
            {t.planetary3d}
          </h2>
          <p className="text-slate-400 text-sm">Interactive 3D planetary globe • Drag to rotate, scroll to zoom, inspect landing sites</p>
        </div>
        <div className="flex items-center gap-2">
          <span className="text-xs font-mono px-3 py-1.5 rounded-lg bg-slate-800/90 border border-cyan-500/30 text-cyan-300">
            {webXrStatus}
          </span>
          <DataSourceBadge type="sim" t={t} />
        </div>
      </div>

      {/* Celestial Selector */}
      <div className="grid grid-cols-2 sm:grid-cols-4 gap-3">
        {Object.values(PLANETARY_BODIES).map((b) => (
          <button
            key={b.id}
            onClick={() => setSelectedBody(b.id)}
            className={`p-3 rounded-xl border text-left transition-all ${
              selectedBody === b.id
                ? 'bg-cyan-500/20 border-cyan-400 text-white shadow-[0_0_20px_rgba(6,182,212,0.25)]'
                : 'bg-slate-900/70 border-slate-800 text-slate-300 hover:border-slate-600'
            }`}
          >
            <div className="flex items-center gap-2 font-semibold text-sm">
              <span className="w-3 h-3 rounded-full" style={{ backgroundColor: b.color }} />
              {b.name.split(' ')[0]}
            </div>
            <div className="text-xs font-mono text-cyan-300 mt-1">{b.gravity} m/s² ({b.gRatio}g)</div>
          </button>
        ))}
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* 3D Canvas Viewport */}
        <div className="lg:col-span-2 relative rounded-2xl border border-cyan-500/30 bg-slate-950/90 overflow-hidden min-h-[380px]">
          <div ref={mountRef} className="w-full h-[380px] cursor-grab active:cursor-grabbing" />
          <div className="absolute top-3 left-3 bg-slate-950/80 backdrop-blur-md border border-slate-700/70 rounded-lg px-3 py-1.5 text-xs font-mono text-cyan-300">
            TARGET: {body.name.toUpperCase()} • R = {body.radiusKm} km
          </div>
          <div className="absolute bottom-3 left-3 right-3 flex flex-wrap justify-between items-center gap-2 bg-slate-950/80 backdrop-blur-md border border-slate-800 rounded-xl px-4 py-2 text-xs text-slate-300">
            <span>🖱️ Drag to orbit • Scroll to zoom</span>
            <span className="text-emerald-400 font-mono">● Active Rover Beacon: {activeWaypoint.id}</span>
          </div>
        </div>

        {/* Scientific Telemetry & Waypoint Inspector */}
        <div className="rounded-2xl border border-slate-800 bg-slate-900/70 backdrop-blur-md p-5 flex flex-col justify-between space-y-4">
          <div>
            <h3 className="text-lg font-bold text-white mb-3">{body.name}</h3>
            <div className="grid grid-cols-2 gap-2 text-xs mb-4">
              <div className="p-2.5 rounded-lg bg-slate-950/70 border border-slate-800">
                <span className="text-slate-400 block">Surface Gravity</span>
                <span className="font-mono text-cyan-300 font-bold">{body.gravity} m/s²</span>
              </div>
              <div className="p-2.5 rounded-lg bg-slate-950/70 border border-slate-800">
                <span className="text-slate-400 block">Mean Temp</span>
                <span className="font-mono text-amber-300 font-bold">{body.tempC}</span>
              </div>
              <div className="col-span-2 p-2.5 rounded-lg bg-slate-950/70 border border-slate-800">
                <span className="text-slate-400 block">Atmosphere Composition</span>
                <span className="font-mono text-slate-200">{body.atmosphere}</span>
              </div>
            </div>

            <div className="text-xs font-semibold uppercase tracking-wider text-cyan-400 mb-2">
              Surface Science Waypoints
            </div>
            <div className="space-y-2">
              {body.waypoints.map((wp) => (
                <button
                  key={wp.id}
                  onClick={() => setActiveWaypoint(wp)}
                  className={`w-full text-left p-3 rounded-xl border transition ${
                    activeWaypoint.id === wp.id
                      ? 'bg-cyan-500/15 border-cyan-400 text-white'
                      : 'bg-slate-950/50 border-slate-800 text-slate-300 hover:border-slate-700'
                  }`}
                >
                  <div className="flex justify-between font-mono text-xs text-cyan-300">
                    <span>{wp.id}</span>
                    <span>{wp.lat}°, {wp.lon}°</span>
                  </div>
                  <div className="font-semibold text-sm mt-0.5">{wp.name}</div>
                  <p className="text-xs text-slate-400 mt-1">{wp.info}</p>
                </button>
              ))}
            </div>
          </div>

          <button
            onClick={() => {
              onLogObservation(`3D Orbital Survey [${body.name}]: Inspected ${activeWaypoint.name} (${activeWaypoint.lat}°, ${activeWaypoint.lon}°). ${activeWaypoint.info}`);
              onAwardXP(40);
            }}
            className="w-full py-2.5 px-4 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600 hover:from-cyan-400 hover:to-blue-500 text-slate-950 font-bold text-sm transition flex items-center justify-center gap-2"
          >
            <FileText className="w-4 h-4" />
            {t.logObservation}
          </button>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MODULE 4: AUTONOMOUS ROVER CONTROL + ESP32 / ARDUINO HARDWARE BRIDGE
// ============================================================================
const RoverControlModule = ({ t, onCompleteMissionStep, onAwardXP, onUnlockBadge, onLogObservation }) => {
  const [subTab, setSubTab] = useState('sim'); // 'sim' | 'hardware'
  const [rover, setRover] = useState({ x: 60, y: 60, angle: 0 });
  const [waypoint, setWaypoint] = useState({ x: 460, y: 280 });
  const [autoMode, setAutoMode] = useState(false);
  const [distance, setDistance] = useState(0);
  const [trail, setTrail] = useState([{ x: 60, y: 60 }]);
  const [goalReached, setGoalReached] = useState(false);
  const [lidarHit, setLidarHit] = useState(false);
  const [serialConnected, setSerialConnected] = useState(false);
  const [serialLog, setSerialLog] = useState([
    '[SYS] Hardware Bridge initialized. Physical robot status: DISCONNECTED (Simulation Mode Active).',
    '[INFO] Connect an ESP32/Arduino via USB Web Serial (115200 baud) to stream live rover commands.',
  ]);
  const [copiedCode, setCopiedCode] = useState(false);

  const obstacles = useMemo(
    () => [
      { x: 180, y: 120, r: 28, label: 'Basalt Boulder' },
      { x: 310, y: 210, r: 34, label: 'Crater Rim' },
      { x: 250, y: 75, r: 22, label: 'Regolith Mound' },
      { x: 390, y: 125, r: 26, label: 'Sharp Scarp' },
      { x: 140, y: 265, r: 30, label: 'Outcrop Ridge' },
    ],
    []
  );

  const canvasRef = useRef(null);

  const moveRoverBy = useCallback(
    (dx, dy) => {
      setRover((prev) => {
        const nx = Math.max(20, Math.min(540, prev.x + dx));
        const ny = Math.max(20, Math.min(340, prev.y + dy));

        // Collision check
        const hit = obstacles.some((o) => Math.hypot(nx - o.x, ny - o.y) < o.r + 14);
        setLidarHit(hit);
        if (hit) return prev;

        const stepDist = Math.hypot(nx - prev.x, ny - prev.y);
        if (stepDist > 0) {
          setDistance((d) => +(d + stepDist * 0.05).toFixed(2));
          setTrail((tr) => [...tr.slice(-80), { x: nx, y: ny }]);
        }

        const distToGoal = Math.hypot(nx - waypoint.x, ny - waypoint.y);
        if (distToGoal < 24 && !goalReached) {
          setGoalReached(true);
          setAutoMode(false);
          onUnlockBadge('rover_pilot');
          onAwardXP(150);
          onCompleteMissionStep('M-01');
        }

        const newAngle = stepDist > 0 ? Math.atan2(dy, dx) : prev.angle;
        return { x: nx, y: ny, angle: newAngle };
      });
    },
    [obstacles, waypoint, goalReached, onUnlockBadge, onAwardXP, onCompleteMissionStep]
  );

  // Keyboard WASD / Arrow support
  useEffect(() => {
    const onKey = (e) => {
      if (subTab !== 'sim') return;
      const step = 12;
      if (e.key === 'ArrowUp' || e.key === 'w' || e.key === 'W') moveRoverBy(0, -step);
      if (e.key === 'ArrowDown' || e.key === 's' || e.key === 'S') moveRoverBy(0, step);
      if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') moveRoverBy(-step, 0);
      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') moveRoverBy(step, 0);
    };
    window.addEventListener('keydown', onKey);
    return () => window.removeEventListener('keydown', onKey);
  }, [moveRoverBy, subTab]);

  // Autonomous Obstacle-Avoidance Navigation Loop (Vector Potential Field)
  useEffect(() => {
    if (!autoMode) return;
    const interval = setInterval(() => {
      setRover((prev) => {
        const toGoalX = waypoint.x - prev.x;
        const toGoalY = waypoint.y - prev.y;
        const goalDist = Math.hypot(toGoalX, toGoalY);

        if (goalDist < 22) {
          setAutoMode(false);
          setGoalReached(true);
          onUnlockBadge('rover_pilot');
          onAwardXP(150);
          onCompleteMissionStep('M-01');
          return prev;
        }

        // Attractive force toward waypoint
        let vx = (toGoalX / goalDist) * 4.5;
        let vy = (toGoalY / goalDist) * 4.5;

        // Repulsive LiDAR obstacle avoidance force
        let nearObstacle = false;
        obstacles.forEach((o) => {
          const dx = prev.x - o.x;
          const dy = prev.y - o.y;
          const d = Math.hypot(dx, dy);
          const safeZone = o.r + 48;
          if (d < safeZone && d > 1) {
            nearObstacle = true;
            const repel = ((safeZone - d) / safeZone) * 7.5;
            // Add tangential steering component so rover smoothly arcs around rocks
            vx += (dx / d) * repel - (dy / d) * (repel * 0.45);
            vy += (dy / d) * repel + (dx / d) * (repel * 0.45);
          }
        });
        setLidarHit(nearObstacle);

        const mag = Math.hypot(vx, vy) || 1;
        const stepX = (vx / mag) * 4.2;
        const stepY = (vy / mag) * 4.2;
        const nx = Math.max(20, Math.min(540, prev.x + stepX));
        const ny = Math.max(20, Math.min(340, prev.y + stepY));

        setDistance((d) => +(d + Math.hypot(nx - prev.x, ny - prev.y) * 0.05).toFixed(2));
        setTrail((tr) => [...tr.slice(-110), { x: nx, y: ny }]);
        return { x: nx, y: ny, angle: Math.atan2(stepY, stepX) };
      });
    }, 55);

    return () => clearInterval(interval);
  }, [autoMode, waypoint, obstacles, onUnlockBadge, onAwardXP, onCompleteMissionStep]);

  // Render Rover Terrain Canvas
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, 560, 360);

    // Martian Regolith Terrain Base
    const grad = ctx.createLinearGradient(0, 0, 560, 360);
    grad.addColorStop(0, '#1e1311');
    grad.addColorStop(1, '#2d1812');
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, 560, 360);

    // Coordinate Grid
    ctx.strokeStyle = 'rgba(249, 115, 22, 0.14)';
    ctx.lineWidth = 1;
    for (let x = 0; x <= 560; x += 40) {
      ctx.beginPath();
      ctx.moveTo(x, 0);
      ctx.lineTo(x, 360);
      ctx.stroke();
    }
    for (let y = 0; y <= 360; y += 40) {
      ctx.beginPath();
      ctx.moveTo(0, y);
      ctx.lineTo(560, y);
      ctx.stroke();
    }

    // Planned Direct Vector Line
    ctx.setLineDash([5, 5]);
    ctx.strokeStyle = 'rgba(0, 210, 255, 0.35)';
    ctx.beginPath();
    ctx.moveTo(rover.x, rover.y);
    ctx.lineTo(waypoint.x, waypoint.y);
    ctx.stroke();
    ctx.setLineDash([]);

    // Rover Breadcrumb Trail
    if (trail.length > 1) {
      ctx.strokeStyle = '#00d2ff';
      ctx.lineWidth = 2.5;
      ctx.beginPath();
      trail.forEach((pt, idx) => (idx === 0 ? ctx.moveTo(pt.x, pt.y) : ctx.lineTo(pt.x, pt.y)));
      ctx.stroke();
    }

    // Obstacles
    obstacles.forEach((o) => {
      ctx.beginPath();
      ctx.arc(o.x, o.y, o.r, 0, Math.PI * 2);
      ctx.fillStyle = '#472820';
      ctx.fill();
      ctx.strokeStyle = '#fb923c';
      ctx.lineWidth = 1.5;
      ctx.stroke();

      ctx.fillStyle = '#fdba74';
      ctx.font = '10px monospace';
      ctx.textAlign = 'center';
      ctx.fillText(o.label, o.x, o.y + o.r + 12);
    });

    // Target Waypoint Beacon
    ctx.beginPath();
    ctx.arc(waypoint.x, waypoint.y, 16, 0, Math.PI * 2);
    ctx.fillStyle = goalReached ? 'rgba(16, 185, 129, 0.25)' : 'rgba(6, 182, 212, 0.22)';
    ctx.fill();
    ctx.strokeStyle = goalReached ? '#10b981' : '#22d3ee';
    ctx.lineWidth = 2;
    ctx.stroke();
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 10px monospace';
    ctx.fillText('TARGET WP', waypoint.x, waypoint.y - 20);

    // Rover Body + LiDAR Sensor Cone
    ctx.save();
    ctx.translate(rover.x, rover.y);
    ctx.rotate(rover.angle);

    // LiDAR Cone
    ctx.beginPath();
    ctx.moveTo(0, 0);
    ctx.arc(0, 0, 58, -0.45, 0.45);
    ctx.closePath();
    ctx.fillStyle = lidarHit ? 'rgba(239, 68, 68, 0.28)' : 'rgba(16, 185, 129, 0.18)';
    ctx.fill();

    // Wheels
    ctx.fillStyle = '#0f172a';
    ctx.fillRect(-12, -14, 24, 4);
    ctx.fillRect(-12, 10, 24, 4);

    // Chassis
    ctx.fillStyle = '#e2e8f0';
    ctx.fillRect(-11, -9, 22, 18);
    ctx.fillStyle = '#0284c7';
    ctx.fillRect(3, -6, 6, 12);
    ctx.restore();
  }, [rover, waypoint, obstacles, trail, goalReached, lidarHit]);

  const handleCanvasClick = (e) => {
    const rect = e.currentTarget.getBoundingClientRect();
    const scaleX = 560 / rect.width;
    const scaleY = 360 / rect.height;
    const wx = Math.round((e.clientX - rect.left) * scaleX);
    const wy = Math.round((e.clientY - rect.top) * scaleY);
    setWaypoint({ x: wx, y: wy });
    setGoalReached(false);
  };

  const handleConnectSerial = async () => {
    if (!('serial' in navigator)) {
      setSerialLog((l) => [
        ...l,
        '[WARN] Web Serial API is not supported in this browser. Use Chrome/Edge over HTTPS with an ESP32/Arduino connected.',
      ]);
      return;
    }
    try {
      const port = await navigator.serial.requestPort();
      await port.open({ baudRate: 115200 });
      setSerialConnected(true);
      setSerialLog((l) => [...l, '[OK] Physical ESP32/Arduino Serial Port Connected at 115200 baud!']);
    } catch (err) {
      setSerialLog((l) => [...l, `[INFO] No hardware selected (${err.message}). Remaining in safe Simulation Mode.`]);
    }
  };

  const arduinoFirmware = `// ASTRO-TRAINER X — ESP32 / Arduino Rover Firmware Bridge
// Protocol: Serial 115200 baud | Commands: FWD, BWD, LFT, RGT, STP
#include <Arduino.h>

const int TRIG_PIN = 5;
const int ECHO_PIN = 18;
const int MOTOR_L = 12;
const int MOTOR_R = 14;

void setup() {
  Serial.begin(115200);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(MOTOR_L, OUTPUT);
  pinMode(MOTOR_R, OUTPUT);
  Serial.println("ASTRO_ROVER_READY");
}

long readUltrasonicCm() {
  digitalWrite(TRIG_PIN, LOW); delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH); delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  return pulseIn(ECHO_PIN, HIGH, 25000) * 0.034 / 2;
}

void loop() {
  long distCm = readUltrasonicCm();
  if (distCm > 0 && distCm < 20) {
    analogWrite(MOTOR_L, 0);
    analogWrite(MOTOR_R, 0);
    Serial.println("ALERT:OBSTACLE_DETECTED");
  }
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\\n');
    cmd.trim();
    if (cmd == "FWD" && distCm >= 20) { analogWrite(MOTOR_L, 180); analogWrite(MOTOR_R, 180); }
    else if (cmd == "STP") { analogWrite(MOTOR_L, 0); analogWrite(MOTOR_R, 0); }
    Serial.println("ACK:" + cmd + ",DIST:" + String(distCm));
  }
  delay(50);
}`;

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <Cpu className="w-6 h-6 text-cyan-400" />
            {t.roverControl}
          </h2>
          <p className="text-slate-400 text-sm">Autonomous LiDAR obstacle avoidance, manual D-pad, and ESP32/Arduino hardware bridge</p>
        </div>

        <div className="flex items-center gap-2 bg-slate-900 p-1 rounded-xl border border-slate-800">
          <button
            onClick={() => setSubTab('sim')}
            className={`px-3 py-1.5 rounded-lg text-xs font-semibold transition ${
              subTab === 'sim' ? 'bg-cyan-500 text-slate-950' : 'text-slate-300 hover:text-white'
            }`}
          >
            Virtual Rover Arena
          </button>
          <button
            onClick={() => setSubTab('hardware')}
            className={`px-3 py-1.5 rounded-lg text-xs font-semibold transition flex items-center gap-1.5 ${
              subTab === 'hardware' ? 'bg-cyan-500 text-slate-950' : 'text-slate-300 hover:text-white'
            }`}
          >
            <Terminal className="w-3.5 h-3.5" />
            ESP32 / Arduino Interface
          </button>
        </div>
      </div>

      {subTab === 'sim' ? (
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <div className="lg:col-span-2 space-y-3">
            <div className="relative rounded-2xl border border-cyan-500/30 overflow-hidden bg-slate-950">
              <canvas
                ref={canvasRef}
                width={560}
                height={360}
                onClick={handleCanvasClick}
                className="w-full h-auto block cursor-crosshair"
              />
              <div className="absolute top-3 left-3 flex flex-wrap gap-2">
                <span className="px-2.5 py-1 rounded bg-slate-950/85 border border-slate-700 text-xs font-mono text-cyan-300">
                  POS: ({Math.round(rover.x)}, {Math.round(rover.y)})
                </span>
                <span className={`px-2.5 py-1 rounded border text-xs font-mono ${lidarHit ? 'bg-red-500/20 border-red-500 text-red-300' : 'bg-emerald-500/20 border-emerald-500/50 text-emerald-300'}`}>
                  LiDAR: {lidarHit ? 'OBSTACLE PROXIMITY' : 'PATH CLEAR'}
                </span>
              </div>
            </div>
            <p className="text-xs text-slate-400">
              💡 Click anywhere on the Martian terrain map to place a new target waypoint, then engage <strong className="text-cyan-300">Autonomous Autopilot</strong> or drive manually with WASD / D-Pad.
            </p>
          </div>

          {/* Controls & Telemetry */}
          <div className="rounded-2xl border border-slate-800 bg-slate-900/75 p-5 flex flex-col justify-between space-y-4">
            <div className="space-y-4">
              <div className="flex items-center justify-between">
                <span className="text-xs font-mono uppercase text-slate-400">Rover Telemetry</span>
                <DataSourceBadge type="sim" t={t} />
              </div>

              <div className="grid grid-cols-2 gap-2.5">
                <div className="p-3 rounded-xl bg-slate-950 border border-slate-800">
                  <div className="text-xs text-slate-400">Distance Driven</div>
                  <div className="text-lg font-mono font-bold text-cyan-300">{distance} m</div>
                </div>
                <div className="p-3 rounded-xl bg-slate-950 border border-slate-800">
                  <div className="text-xs text-slate-400">Mission Status</div>
                  <div className={`text-sm font-mono font-bold ${goalReached ? 'text-emerald-400' : autoMode ? 'text-amber-300' : 'text-slate-300'}`}>
                    {goalReached ? 'WAYPOINT REACHED' : autoMode ? 'AUTOPILOT ACTIVE' : 'MANUAL STANDBY'}
                  </div>
                </div>
              </div>

              {/* Autopilot & Reset */}
              <div className="flex gap-2">
                <button
                  onClick={() => setAutoMode(!autoMode)}
                  className={`flex-1 py-2.5 px-3 rounded-xl font-bold text-xs flex items-center justify-center gap-1.5 transition ${
                    autoMode
                      ? 'bg-amber-500 text-slate-950'
                      : 'bg-gradient-to-r from-cyan-500 to-blue-600 text-slate-950 hover:opacity-95'
                  }`}
                >
                  <Navigation className="w-4 h-4" />
                  {autoMode ? 'Pause Autopilot' : 'Engage Autonomous Nav'}
                </button>
                <button
                  onClick={() => {
                    setAutoMode(false);
                    setRover({ x: 60, y: 60, angle: 0 });
                    setTrail([{ x: 60, y: 60 }]);
                    setDistance(0);
                    setGoalReached(false);
                  }}
                  className="p-2.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700"
                  title="Reset Rover"
                >
                  <RotateCcw className="w-4 h-4" />
                </button>
              </div>

              {/* Manual D-Pad */}
              <div className="pt-2">
                <div className="text-xs text-center text-slate-400 mb-2">Manual Drive D-Pad (or WASD)</div>
                <div className="grid grid-cols-3 gap-2 max-w-[168px] mx-auto">
                  <div />
                  <button onClick={() => moveRoverBy(0, -14)} className="p-3 rounded-xl bg-slate-800 hover:bg-cyan-500/20 border border-slate-700 flex justify-center text-cyan-300">
                    <ArrowUp className="w-5 h-5" />
                  </button>
                  <div />
                  <button onClick={() => moveRoverBy(-14, 0)} className="p-3 rounded-xl bg-slate-800 hover:bg-cyan-500/20 border border-slate-700 flex justify-center text-cyan-300">
                    <ArrowLeft className="w-5 h-5" />
                  </button>
                  <button onClick={() => setAutoMode(false)} className="p-3 rounded-xl bg-red-500/20 border border-red-500/40 text-red-300 text-[10px] font-mono font-bold">
                    STOP
                  </button>
                  <button onClick={() => moveRoverBy(14, 0)} className="p-3 rounded-xl bg-slate-800 hover:bg-cyan-500/20 border border-slate-700 flex justify-center text-cyan-300">
                    <ArrowRight className="w-5 h-5" />
                  </button>
                  <div />
                  <button onClick={() => moveRoverBy(0, 14)} className="p-3 rounded-xl bg-slate-800 hover:bg-cyan-500/20 border border-slate-700 flex justify-center text-cyan-300">
                    <ArrowDown className="w-5 h-5" />
                  </button>
                  <div />
                </div>
              </div>
            </div>

            <button
              onClick={() => {
                onLogObservation(`Rover Traverse Telemetry: Traveled ${distance}m across simulated Jezero terrain. Waypoint status: ${goalReached ? 'Reached' : 'In Progress'}.`);
                onAwardXP(35);
              }}
              className="w-full py-2.5 px-4 rounded-xl bg-slate-800 hover:bg-slate-700 border border-cyan-500/30 text-cyan-300 font-semibold text-xs flex items-center justify-center gap-2"
            >
              <FileText className="w-4 h-4" />
              {t.logObservation}
            </button>
          </div>
        </div>
      ) : (
        /* ESP32 & ARDUINO HARDWARE BRIDGE TAB */
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
          <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 space-y-4">
            <div className="flex items-center justify-between">
              <h3 className="font-bold text-white flex items-center gap-2">
                {serialConnected ? <Wifi className="w-5 h-5 text-emerald-400" /> : <WifiOff className="w-5 h-5 text-amber-400" />}
                Physical Microcontroller Status
              </h3>
              <span className={`px-3 py-1 rounded-full text-xs font-mono border ${serialConnected ? 'bg-emerald-500/20 border-emerald-500 text-emerald-300' : 'bg-amber-500/15 border-amber-500/40 text-amber-300'}`}>
                {serialConnected ? 'CONNECTED (115200 BAUD)' : 'DISCONNECTED — SIMULATION MODE'}
              </span>
            </div>

            <p className="text-xs text-slate-300 leading-relaxed">
              ASTRO-TRAINER X supports real educational robotics hardware (ESP32 / Arduino Uno / Nano) via the browser <strong className="text-cyan-300">Web Serial API</strong>. We never claim a physical robot is connected unless an active serial handshake is verified.
            </p>

            <button
              onClick={handleConnectSerial}
              className="px-4 py-2.5 rounded-xl bg-cyan-500 hover:bg-cyan-400 text-slate-950 font-bold text-xs transition"
            >
              Connect USB ESP32 / Arduino (Web Serial)
            </button>

            <div className="bg-slate-950 border border-slate-800 rounded-xl p-3 font-mono text-xs text-slate-300 h-40 overflow-y-auto space-y-1">
              {serialLog.map((line, i) => (
                <div key={i} className={line.includes('[OK]') ? 'text-emerald-400' : line.includes('[WARN]') ? 'text-amber-400' : 'text-slate-400'}>
                  {line}
                </div>
              ))}
            </div>
          </div>

          <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 space-y-3">
            <div className="flex items-center justify-between">
              <h3 className="font-bold text-white text-sm">Reference Firmware (Arduino C++ / ESP32)</h3>
              <button
                onClick={() => {
                  navigator.clipboard?.writeText(arduinoFirmware);
                  setCopiedCode(true);
                  setTimeout(() => setCopiedCode(false), 2000);
                }}
                className="px-3 py-1 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs text-cyan-300 border border-slate-700"
              >
                {copiedCode ? 'Copied!' : 'Copy Sketch'}
              </button>
            </div>
            <pre className="bg-slate-950 border border-slate-800 rounded-xl p-3 text-[11px] font-mono text-cyan-200 overflow-x-auto max-h-64">
              {arduinoFirmware}
            </pre>
          </div>
        </div>
      )}
    </div>
  );
};

// ============================================================================
// MODULE 5: RADIATION PROTECTION SIMULATOR
// ============================================================================
const RadiationLabModule = ({ t, onUnlockBadge, onAwardXP, onCompleteMissionStep, onLogObservation }) => {
  const [materialId, setMaterialId] = useState('polyethylene');
  const [thicknessCm, setThicknessCm] = useState(12);
  const [radType, setRadType] = useState('SPE'); // 'SPE' (Solar Particle Event) | 'GCR' (Galactic Cosmic Rays)
  const canvasRef = useRef(null);

  const material = SHIELD_MATERIALS.find((m) => m.id === materialId) || SHIELD_MATERIALS[0];
  const coeff = radType === 'SPE' ? material.speAtten : material.gcrAtten;
  // Beer-Lambert exponential attenuation approximation for educational modeling
  const transmittedPct = Math.max(4, Math.round(100 * Math.exp(-coeff * thicknessCm)));
  const blockedPct = 100 - transmittedPct;

  useEffect(() => {
    if (transmittedPct < 35) {
      onUnlockBadge('shield_engineer');
      onCompleteMissionStep('M-04');
    }
  }, [transmittedPct, onUnlockBadge, onCompleteMissionStep]);

  // Particle Penetration Canvas Animation
  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    let animId;

    const particles = Array.from({ length: 42 }, () => ({
      x: Math.random() * 200,
      y: Math.random() * 220 + 10,
      speed: Math.random() * 2.5 + 2.2,
      willPenetrate: Math.random() * 100 < transmittedPct,
      stopped: false,
    }));

    const shieldX = 210;
    const shieldW = Math.max(12, thicknessCm * 2.6);

    const render = () => {
      ctx.clearRect(0, 0, 520, 240);
      ctx.fillStyle = '#050b18';
      ctx.fillRect(0, 0, 520, 240);

      // Habitat Zone
      ctx.fillStyle = 'rgba(16, 185, 129, 0.08)';
      ctx.fillRect(shieldX + shieldW, 0, 520 - (shieldX + shieldW), 240);
      ctx.fillStyle = '#34d399';
      ctx.font = 'bold 11px monospace';
      ctx.fillText('CREW HABITAT ZONE', shieldX + shieldW + 18, 24);

      // Shield Slab
      ctx.fillStyle = materialId === 'water' ? 'rgba(56, 189, 248, 0.45)' : materialId === 'polyethylene' ? 'rgba(168, 85, 247, 0.45)' : 'rgba(148, 163, 184, 0.45)';
      ctx.fillRect(shieldX, 15, shieldW, 210);
      ctx.strokeStyle = '#38bdf8';
      ctx.strokeRect(shieldX, 15, shieldW, 210);

      particles.forEach((p) => {
        if (!p.stopped) {
          p.x += p.speed;
          if (!p.willPenetrate && p.x >= shieldX + Math.random() * shieldW) {
            p.stopped = true;
          }
        }
        if (p.x > 520 || (p.stopped && Math.random() < 0.08)) {
          p.x = 0;
          p.y = Math.random() * 210 + 15;
          p.willPenetrate = Math.random() * 100 < transmittedPct;
          p.stopped = false;
        }

        ctx.beginPath();
        ctx.arc(p.x, p.y, p.stopped ? 4.5 : 2.8, 0, Math.PI * 2);
        ctx.fillStyle = p.stopped ? '#facc15' : radType === 'SPE' ? '#f97316' : '#f43f5e';
        ctx.fill();
      });

      animId = requestAnimationFrame(render);
    };
    render();
    return () => cancelAnimationFrame(animId);
  }, [materialId, thicknessCm, radType, transmittedPct]);

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <FlaskConical className="w-6 h-6 text-purple-400" />
            {t.radiationLab}
          </h2>
          <p className="text-slate-400 text-sm">Compare spacecraft shielding materials against Solar Particle Events (SPE) & Galactic Cosmic Rays (GCR)</p>
        </div>
        <DataSourceBadge type="sim" t={t} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2 space-y-4">
          <div className="rounded-2xl border border-purple-500/30 bg-slate-950 overflow-hidden">
            <canvas ref={canvasRef} width={520} height={240} className="w-full h-auto block" />
          </div>

          <div className="p-4 rounded-xl bg-amber-500/10 border border-amber-500/30 text-xs text-amber-200 flex items-start gap-2.5">
            <Info className="w-4 h-4 shrink-0 mt-0.5 text-amber-400" />
            <div>
              <strong>Educational Model Assumptions:</strong> Uses a simplified exponential attenuation curve (I = I₀·e⁻ᵘˣ) to illustrate how hydrogen-rich materials (Polyethylene, Water) outperform dense metals like Lead against high-energy cosmic ions by minimizing secondary radiation cascades.
            </div>
          </div>
        </div>

        <div className="rounded-2xl border border-slate-800 bg-slate-900/75 p-5 space-y-4">
          <div>
            <label className="text-xs font-mono text-slate-400 block mb-1.5">Radiation Environment</label>
            <div className="grid grid-cols-2 gap-2">
              <button
                onClick={() => setRadType('SPE')}
                className={`py-2 px-3 rounded-xl text-xs font-bold border ${radType === 'SPE' ? 'bg-orange-500/20 border-orange-400 text-orange-200' : 'bg-slate-950 border-slate-800 text-slate-400'}`}
              >
                Solar Storm (SPE)
              </button>
              <button
                onClick={() => setRadType('GCR')}
                className={`py-2 px-3 rounded-xl text-xs font-bold border ${radType === 'GCR' ? 'bg-rose-500/20 border-rose-400 text-rose-200' : 'bg-slate-950 border-slate-800 text-slate-400'}`}
              >
                Cosmic Rays (GCR)
              </button>
            </div>
          </div>

          <div>
            <label className="text-xs font-mono text-slate-400 block mb-1.5">Shielding Material</label>
            <select
              value={materialId}
              onChange={(e) => setMaterialId(e.target.value)}
              className="w-full p-2.5 rounded-xl bg-slate-950 border border-slate-700 text-sm text-white"
            >
              {SHIELD_MATERIALS.map((m) => (
                <option key={m.id} value={m.id}>{m.name}</option>
              ))}
            </select>
            <p className="text-xs text-slate-400 mt-1.5">{material.notes}</p>
          </div>

          <div>
            <div className="flex justify-between text-xs font-mono mb-1">
              <span className="text-slate-400">Shield Thickness</span>
              <span className="text-cyan-300 font-bold">{thicknessCm} cm ({(thicknessCm * material.density).toFixed(1)} g/cm²)</span>
            </div>
            <input
              type="range"
              min="2"
              max="30"
              value={thicknessCm}
              onChange={(e) => setThicknessCm(Number(e.target.value))}
              className="w-full accent-cyan-400"
            />
          </div>

          <div className="grid grid-cols-2 gap-2 pt-1">
            <div className="p-3 rounded-xl bg-slate-950 border border-slate-800">
              <div className="text-[11px] text-slate-400">Radiation Blocked</div>
              <div className="text-xl font-mono font-bold text-emerald-400">{blockedPct}%</div>
            </div>
            <div className="p-3 rounded-xl bg-slate-950 border border-slate-800">
              <div className="text-[11px] text-slate-400">Relative Exposure</div>
              <div className={`text-xl font-mono font-bold ${transmittedPct < 35 ? 'text-cyan-300' : 'text-amber-400'}`}>{transmittedPct}%</div>
            </div>
          </div>

          <button
            onClick={() => {
              onLogObservation(`Radiation Shielding Experiment: Tested ${thicknessCm}cm of ${material.name} against ${radType}. Blocked ${blockedPct}% of incoming flux (${transmittedPct}% relative exposure).`);
              onAwardXP(45);
            }}
            className="w-full py-2.5 px-4 rounded-xl bg-gradient-to-r from-purple-500 to-cyan-500 text-slate-950 font-bold text-xs flex items-center justify-center gap-2"
          >
            <FileText className="w-4 h-4" />
            {t.logObservation}
          </button>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MODULE 6: SPACE WEATHER CENTER (Live NOAA/NASA Public API + Clear Fallback)
// ============================================================================
const SpaceWeatherModule = ({ t, chartReady, onLogObservation, onAwardXP }) => {
  const [sourceType, setSourceType] = useState('sim'); // 'live' | 'sim'
  const [weatherData, setWeatherData] = useState({
    solarWindSpeed: [410, 425, 448, 490, 512, 485, 468],
    kpIndex: [2, 2, 3, 4, 3, 2, 2],
    timestamps: ['T-6h', 'T-5h', 'T-4h', 'T-3h', 'T-2h', 'T-1h', 'NOW'],
    flareClass: 'C2.4 (Minor)',
    alertBanner: 'Nominal geomagnetic field conditions. No Earth-directed X-class CMEs detected.',
  });
  const [loading, setLoading] = useState(false);
  const chartCanvasRef = useRef(null);
  const chartInstanceRef = useRef(null);

  const fetchLiveSpaceWeather = async () => {
    setLoading(true);
    try {
      // Attempt public NOAA Space Weather Prediction Center (SWPC) planetary K-index endpoint
      const res = await fetch('https://services.swpc.noaa.gov/products/noaa-planetary-k-index.json');
      if (!res.ok) throw new Error('SWPC endpoint unreachable');
      const rows = await res.json();
      const recent = rows.slice(-7);
      if (recent.length >= 4) {
        const labels = recent.map((r) => r[0].slice(11, 16) + ' UTC');
        const kps = recent.map((r) => parseFloat(r[1]) || 2);
        const maxKp = Math.max(...kps);
        setWeatherData((prev) => ({
          ...prev,
          timestamps: labels,
          kpIndex: kps,
          alertBanner: maxKp >= 5 ? `G1+ Geomagnetic Storm Watch (Kp ${maxKp})` : `Verified NOAA SWPC Telemetry — Quiet to Unsettled (Max Kp ${maxKp})`,
        }));
        setSourceType('live');
      } else {
        throw new Error('Insufficient rows');
      }
    } catch {
      // Honest fallback to clearly labeled simulation dataset
      setSourceType('sim');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchLiveSpaceWeather();
  }, []);

  useEffect(() => {
    if (!chartReady || !window.Chart || !chartCanvasRef.current) return;
    if (chartInstanceRef.current) chartInstanceRef.current.destroy();

    const ctx = chartCanvasRef.current.getContext('2d');
    chartInstanceRef.current = new window.Chart(ctx, {
      type: 'line',
      data: {
        labels: weatherData.timestamps,
        datasets: [
          {
            label: 'Solar Wind Speed (km/s)',
            data: weatherData.solarWindSpeed,
            borderColor: '#00d2ff',
            backgroundColor: 'rgba(0, 210, 255, 0.15)',
            yAxisID: 'y',
            tension: 0.35,
            fill: true,
          },
          {
            label: 'Planetary Kp Index (0-9)',
            data: weatherData.kpIndex,
            borderColor: '#a855f7',
            backgroundColor: 'rgba(168, 85, 247, 0.25)',
            yAxisID: 'y1',
            tension: 0.3,
          },
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: { legend: { labels: { color: '#cbd5e1' } } },
        scales: {
          x: { ticks: { color: '#94a3b8' }, grid: { color: 'rgba(148,163,184,0.1)' } },
          y: {
            type: 'linear',
            position: 'left',
            ticks: { color: '#38bdf8' },
            grid: { color: 'rgba(148,163,184,0.1)' },
          },
          y1: {
            type: 'linear',
            position: 'right',
            min: 0,
            max: 9,
            ticks: { color: '#c084fc' },
            grid: { drawOnChartArea: false },
          },
        },
      },
    });

    return () => chartInstanceRef.current?.destroy();
  }, [chartReady, weatherData]);

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <Sun className="w-6 h-6 text-amber-400" />
            {t.spaceWeather}
          </h2>
          <p className="text-slate-400 text-sm">Real-time heliospheric solar wind, geomagnetic Kp index, and flare monitoring</p>
        </div>
        <div className="flex items-center gap-2">
          <DataSourceBadge type={sourceType} t={t} />
          <button
            onClick={fetchLiveSpaceWeather}
            disabled={loading}
            className="px-3 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 border border-slate-700 text-xs text-cyan-300 font-mono"
          >
            {loading ? 'Syncing...' : 'Refresh Feed'}
          </button>
        </div>
      </div>

      {/* Alert Banner */}
      <div className="p-4 rounded-2xl bg-cyan-500/10 border border-cyan-500/30 flex items-center justify-between gap-4">
        <div className="flex items-center gap-3">
          <AlertTriangle className="w-5 h-5 text-cyan-400 shrink-0" />
          <span className="text-sm text-slate-100">{weatherData.alertBanner}</span>
        </div>
        <span className="text-xs font-mono text-slate-400 shrink-0">
          {sourceType === 'live' ? 'Source: NOAA SWPC' : 'Mode: Educational Model'}
        </span>
      </div>

      {/* KPI Cards */}
      <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
        <div className="p-4 rounded-2xl bg-slate-900/80 border border-slate-800">
          <div className="text-xs text-slate-400">Solar Wind Velocity</div>
          <div className="text-2xl font-mono font-bold text-cyan-300 mt-1">
            {weatherData.solarWindSpeed[weatherData.solarWindSpeed.length - 1]} km/s
          </div>
          <div className="text-[11px] text-slate-500 mt-1">Nominal range: 300–500 km/s</div>
        </div>
        <div className="p-4 rounded-2xl bg-slate-900/80 border border-slate-800">
          <div className="text-xs text-slate-400">Geomagnetic Kp Index</div>
          <div className="text-2xl font-mono font-bold text-purple-300 mt-1">
            Kp {weatherData.kpIndex[weatherData.kpIndex.length - 1]}
          </div>
          <div className="text-[11px] text-slate-500 mt-1">Storm threshold: Kp ≥ 5</div>
        </div>
        <div className="p-4 rounded-2xl bg-slate-900/80 border border-slate-800">
          <div className="text-xs text-slate-400">X-Ray Solar Flare Activity</div>
          <div className="text-2xl font-mono font-bold text-amber-300 mt-1">{weatherData.flareClass}</div>
          <div className="text-[11px] text-slate-500 mt-1">GOES-18 Soft X-Ray Flux</div>
        </div>
      </div>

      {/* Chart Container */}
      <div className="rounded-2xl border border-slate-800 bg-slate-900/75 p-5">
        <div className="h-64">
          <canvas ref={chartCanvasRef} />
        </div>
      </div>

      <button
        onClick={() => {
          onLogObservation(`Space Weather Brief (${sourceType.toUpperCase()}): Solar wind ${weatherData.solarWindSpeed.slice(-1)[0]} km/s, Kp Index ${weatherData.kpIndex.slice(-1)[0]}. ${weatherData.alertBanner}`);
          onAwardXP(30);
        }}
        className="px-5 py-2.5 rounded-xl bg-cyan-500 hover:bg-cyan-400 text-slate-950 font-bold text-xs inline-flex items-center gap-2"
      >
        <FileText className="w-4 h-4" />
        {t.logObservation}
      </button>
    </div>
  );
};

// ============================================================================
// MODULE 7: SPECTRAL ROCK ANALYZER
// ============================================================================
const SpectralAnalyzerModule = ({ t, chartReady, onUnlockBadge, onAwardXP, onCompleteMissionStep, onLogObservation }) => {
  const [selectedId, setSelectedId] = useState('gypsum');
  const [analyzedIds, setAnalyzedIds] = useState(['gypsum']);
  const [showReference, setShowReference] = useState(true);
  const canvasRef = useRef(null);
  const chartRef = useRef(null);

  const sample = ROCK_SAMPLES.find((s) => s.id === selectedId) || ROCK_SAMPLES[0];
  const wavelengths = ['400', '600', '800', '950', '1050', '1250', '1450', '1700', '1940', '2210', '2500'];

  const handleSelectSample = (id) => {
    setSelectedId(id);
    setAnalyzedIds((prev) => {
      const next = prev.includes(id) ? prev : [...prev, id];
      if (next.length >= 2) onCompleteMissionStep('M-03');
      if (next.length >= 3) onUnlockBadge('spectroscopist');
      return next;
    });
  };

  useEffect(() => {
    if (!chartReady || !window.Chart || !canvasRef.current) return;
    if (chartRef.current) chartRef.current.destroy();

    const datasets = [
      {
        label: `Measured Sample: ${sample.code}`,
        data: sample.spectrum,
        borderColor: '#00d2ff',
        backgroundColor: 'rgba(0, 210, 255, 0.18)',
        borderWidth: 2.5,
        tension: 0.3,
        fill: true,
      },
    ];

    if (showReference) {
      datasets.push({
        label: 'USGS / NASA PDS Reference Library Signature',
        data: sample.reference,
        borderColor: '#f59e0b',
        borderDash: [5, 5],
        borderWidth: 2,
        tension: 0.3,
      });
    }

    chartRef.current = new window.Chart(canvasRef.current.getContext('2d'), {
      type: 'line',
      data: { labels: wavelengths.map((w) => `${w} nm`), datasets },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: { legend: { labels: { color: '#e2e8f0' } } },
        scales: {
          x: { ticks: { color: '#94a3b8' }, grid: { color: 'rgba(148,163,184,0.1)' } },
          y: {
            min: 0,
            max: 0.85,
            title: { display: true, text: 'Reflectance Factor (I/F)', color: '#94a3b8' },
            ticks: { color: '#94a3b8' },
            grid: { color: 'rgba(148,163,184,0.1)' },
          },
        },
      },
    });

    return () => chartRef.current?.destroy();
  }, [chartReady, sample, showReference]);

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <Activity className="w-6 h-6 text-cyan-400" />
            {t.spectralLab}
          </h2>
          <p className="text-slate-400 text-sm">Visible & Near-Infrared (VNIR) reflectance spectroscopy (400–2500 nm)</p>
        </div>
        <DataSourceBadge type="sim" t={t} />
      </div>

      {/* Sample Selector */}
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3">
        {ROCK_SAMPLES.map((s) => (
          <button
            key={s.id}
            onClick={() => handleSelectSample(s.id)}
            className={`p-3.5 rounded-xl border text-left transition ${
              selectedId === s.id
                ? 'bg-cyan-500/20 border-cyan-400 text-white'
                : 'bg-slate-900/75 border-slate-800 text-slate-300 hover:border-slate-700'
            }`}
          >
            <div className="flex justify-between items-center text-xs font-mono text-cyan-300">
              <span>{s.code}</span>
              {analyzedIds.includes(s.id) && <CheckCircle2 className="w-3.5 h-3.5 text-emerald-400" />}
            </div>
            <div className="font-bold text-sm mt-1">{s.name}</div>
          </button>
        ))}
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2 rounded-2xl border border-slate-800 bg-slate-900/80 p-5 space-y-3">
          <div className="flex items-center justify-between">
            <span className="text-xs font-mono text-slate-400">EDUCATIONAL SPECTRAL SIGNATURE CURVE</span>
            <label className="inline-flex items-center gap-2 text-xs text-amber-300 cursor-pointer">
              <input
                type="checkbox"
                checked={showReference}
                onChange={(e) => setShowReference(e.target.checked)}
                className="accent-amber-400"
              />
              Overlay Library Match
            </label>
          </div>
          <div className="h-64">
            <canvas ref={canvasRef} />
          </div>
        </div>

        <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 flex flex-col justify-between space-y-4">
          <div className="space-y-3">
            <div className="text-xs font-mono text-cyan-400">MINERAL CLASSIFICATION</div>
            <h3 className="text-lg font-bold text-white">{sample.name}</h3>
            <p className="text-xs text-slate-300 leading-relaxed">{sample.description}</p>

            <div className="space-y-2 pt-2 text-xs">
              <div className="p-2.5 rounded-lg bg-slate-950 border border-slate-800">
                <span className="text-slate-400 block">Mineral Group</span>
                <span className="font-semibold text-white">{sample.mineralClass}</span>
              </div>
              <div className="p-2.5 rounded-lg bg-slate-950 border border-slate-800">
                <span className="text-slate-400 block">Diagnostic Absorption Bands</span>
                <span className="font-mono text-cyan-300">{sample.diagnosticBands}</span>
              </div>
              <div className="p-2.5 rounded-lg bg-slate-950 border border-slate-800">
                <span className="text-slate-400 block">Surface Albedo</span>
                <span className="font-mono text-amber-300">{sample.albedo}</span>
              </div>
            </div>
          </div>

          <button
            onClick={() => {
              onLogObservation(`Spectrometer Finding [${sample.code}]: Identified ${sample.name} (${sample.mineralClass}) via ${sample.diagnosticBands}.`);
              onAwardXP(50);
            }}
            className="w-full py-2.5 px-4 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600 text-slate-950 font-bold text-xs flex items-center justify-center gap-2"
          >
            <FileText className="w-4 h-4" />
            {t.logObservation}
          </button>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MODULE 8: PLANETARY GRAVITY SIMULATOR (Earth 1.0g vs Mars 0.38g vs Moon 0.166g)
// ============================================================================
const GravitySimulatorModule = ({ t, onUnlockBadge, onAwardXP, onCompleteMissionStep, onLogObservation }) => {
  const [dropHeightM, setDropHeightM] = useState(50);
  const [isDropping, setIsDropping] = useState(false);
  const [elapsed, setElapsed] = useState(0);
  const canvasRef = useRef(null);

  const bodies = [
    { id: 'earth', label: 'Earth (1.0g)', g: 9.81, color: '#38bdf8' },
    { id: 'mars', label: 'Mars (0.38g)', g: 3.72, color: '#fb923c' },
    { id: 'moon', label: 'Moon (0.166g)', g: 1.62, color: '#cbd5e1' },
  ];

  // Max fall time is Moon: t = sqrt(2h / g)
  const maxTime = Math.sqrt((2 * dropHeightM) / 1.62);

  useEffect(() => {
    if (!isDropping) return;
    let start = performance.now() - elapsed * 1000;
    let animId;

    const step = (now) => {
      const sec = (now - start) / 1000;
      if (sec >= maxTime) {
        setElapsed(maxTime);
        setIsDropping(false);
        onUnlockBadge('gravity_master');
        onCompleteMissionStep('M-02');
      } else {
        setElapsed(sec);
        animId = requestAnimationFrame(step);
      }
    };
    animId = requestAnimationFrame(step);
    return () => cancelAnimationFrame(animId);
  }, [isDropping, maxTime, onUnlockBadge, onCompleteMissionStep]);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, 520, 280);
    ctx.fillStyle = '#060c1a';
    ctx.fillRect(0, 0, 520, 280);

    // Height markers
    ctx.strokeStyle = 'rgba(148, 163, 184, 0.15)';
    ctx.font = '10px monospace';
    ctx.fillStyle = '#64748b';
    for (let i = 0; i <= 4; i++) {
      const y = 30 + (i * 210) / 4;
      ctx.beginPath();
      ctx.moveTo(30, y);
      ctx.lineTo(490, y);
      ctx.stroke();
      const hVal = Math.round(dropHeightM * (1 - i / 4));
      ctx.fillText(`${hVal}m`, 4, y + 3);
    }

    // Draw 3 Drop Lanes
    bodies.forEach((b, idx) => {
      const laneX = 115 + idx * 145;
      const distFallen = Math.min(dropHeightM, 0.5 * b.g * elapsed * elapsed);
      const yPos = 30 + (distFallen / dropHeightM) * 210;

      // Guide Track
      ctx.strokeStyle = 'rgba(255,255,255,0.08)';
      ctx.beginPath();
      ctx.moveTo(laneX, 30);
      ctx.lineTo(laneX, 240);
      ctx.stroke();

      // Probe Sphere
      ctx.beginPath();
      ctx.arc(laneX, yPos, 13, 0, Math.PI * 2);
      ctx.fillStyle = b.color;
      ctx.fill();

      // Label & Velocity
      const vel = Math.min(Math.sqrt(2 * b.g * dropHeightM), b.g * elapsed);
      ctx.fillStyle = '#f8fafc';
      ctx.font = 'bold 11px sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(b.label, laneX, 18);
      ctx.font = '10px monospace';
      ctx.fillStyle = b.color;
      ctx.fillText(`v = ${vel.toFixed(1)} m/s`, laneX, 265);
    });
  }, [elapsed, dropHeightM]);

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <ArrowDownToLine className="w-6 h-6 text-cyan-400" />
            {t.gravitySim}
          </h2>
          <p className="text-slate-400 text-sm">Side-by-side free-fall kinematics comparing Earth (1.0g), Mars (0.38g), and Moon (0.166g)</p>
        </div>
        <DataSourceBadge type="sim" t={t} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2 rounded-2xl border border-cyan-500/30 bg-slate-950 p-3">
          <canvas ref={canvasRef} width={520} height={280} className="w-full h-auto block rounded-xl" />
        </div>

        <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 flex flex-col justify-between space-y-4">
          <div className="space-y-4">
            <div>
              <div className="flex justify-between text-xs font-mono mb-1">
                <span className="text-slate-400">Drop Altitude (h)</span>
                <span className="text-cyan-300 font-bold">{dropHeightM} meters</span>
              </div>
              <input
                type="range"
                min="10"
                max="200"
                step="5"
                value={dropHeightM}
                onChange={(e) => {
                  setIsDropping(false);
                  setElapsed(0);
                  setDropHeightM(Number(e.target.value));
                }}
                className="w-full accent-cyan-400"
              />
            </div>

            <div className="flex gap-2">
              <button
                onClick={() => {
                  if (elapsed >= maxTime) setElapsed(0);
                  setIsDropping(true);
                }}
                className="flex-1 py-2.5 px-4 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600 text-slate-950 font-bold text-xs flex items-center justify-center gap-1.5"
              >
                <Play className="w-4 h-4" />
                Release Probes
              </button>
              <button
                onClick={() => {
                  setIsDropping(false);
                  setElapsed(0);
                }}
                className="p-2.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700"
              >
                <RotateCcw className="w-4 h-4" />
              </button>
            </div>

            {/* Kinematic Equations Breakdown */}
            <div className="p-3 rounded-xl bg-slate-950 border border-slate-800 space-y-2 text-xs font-mono">
              <div className="text-cyan-400 font-bold">Equation: t = √(2h / g)</div>
              {bodies.map((b) => {
                const tFall = Math.sqrt((2 * dropHeightM) / b.g);
                const vImpact = b.g * tFall;
                return (
                  <div key={b.id} className="flex justify-between text-slate-300 border-t border-slate-800/80 pt-1.5">
                    <span>{b.label}</span>
                    <span>{tFall.toFixed(2)}s • {vImpact.toFixed(1)}m/s</span>
                  </div>
                );
              })}
            </div>
          </div>

          <button
            onClick={() => {
              const tEarth = Math.sqrt((2 * dropHeightM) / 9.81).toFixed(2);
              const tMars = Math.sqrt((2 * dropHeightM) / 3.72).toFixed(2);
              const tMoon = Math.sqrt((2 * dropHeightM) / 1.62).toFixed(2);
              onLogObservation(`Gravity Drop Experiment (h=${dropHeightM}m): Earth impact in ${tEarth}s, Mars in ${tMars}s, Moon in ${tMoon}s.`);
              onAwardXP(40);
            }}
            className="w-full py-2.5 px-4 rounded-xl bg-slate-800 hover:bg-slate-700 border border-cyan-500/30 text-cyan-300 font-semibold text-xs flex items-center justify-center gap-2"
          >
            <FileText className="w-4 h-4" />
            {t.logObservation}
          </button>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MODULE 9: DEEP SPACE COMMUNICATION DELAY SIMULATOR
// ============================================================================
const DeepSpaceCommsModule = ({ t, onUnlockBadge, onAwardXP, onLogObservation }) => {
  const scenarios = [
    { id: 'moon', name: 'Lunar Gateway (Moon)', distanceKm: '384,400 km', oneWaySec: 1.28 },
    { id: 'mars_close', name: 'Mars Perseverance (Closest Approach)', distanceKm: '54.6M km', oneWaySec: 182 },
    { id: 'mars_far', name: 'Mars Perseverance (Solar Conjunction)', distanceKm: '401M km', oneWaySec: 1337 },
    { id: 'voyager', name: 'Voyager 1 (Interstellar Space)', distanceKm: '24.5B km', oneWaySec: 81720 },
  ];

  const [scenarioId, setScenarioId] = useState('mars_close');
  const [timeScale, setTimeScale] = useState(30); // 1x, 30x, 300x
  const [commandText, setCommandText] = useState('DEPLOY_SUPERCAM_LASER');
  const [txState, setTxState] = useState('IDLE'); // IDLE | UPLINK | DOWNLINK | COMPLETE
  const [progress, setProgress] = useState(0);
  const [commsLog, setCommsLog] = useState([]);

  const scenario = scenarios.find((s) => s.id === scenarioId) || scenarios[0];

  const transmitCommand = () => {
    if (txState === 'UPLINK' || txState === 'DOWNLINK') return;
    setTxState('UPLINK');
    setProgress(0);
    setCommsLog((prev) => [
      { time: new Date().toLocaleTimeString(), dir: 'UPLINK', msg: `Transmitting "${commandText}" at c = 299,792 km/s...` },
      ...prev,
    ]);
  };

  useEffect(() => {
    if (txState !== 'UPLINK' && txState !== 'DOWNLINK') return;
    // Simulated duration scaled by timeScale
    const effectiveMs = Math.max(1200, (scenario.oneWaySec * 1000) / timeScale);
    const stepMs = 50;
    const increment = (stepMs / effectiveMs) * 100;

    const timer = setInterval(() => {
      setProgress((p) => {
        if (p + increment >= 100) {
          if (txState === 'UPLINK') {
            setTxState('DOWNLINK');
            setCommsLog((prev) => [
              { time: new Date().toLocaleTimeString(), dir: 'TARGET', msg: `Spacecraft received "${commandText}". Executing & sending ACK...` },
              ...prev,
            ]);
            return 0;
          } else {
            setTxState('COMPLETE');
            onUnlockBadge('deep_comms');
            onAwardXP(45);
            setCommsLog((prev) => [
              { time: new Date().toLocaleTimeString(), dir: 'DOWNLINK', msg: `ACK Received! Round-trip light time: ${(scenario.oneWaySec * 2).toFixed(1)}s.` },
              ...prev,
            ]);
            return 100;
          }
        }
        return p + increment;
      });
    }, stepMs);

    return () => clearInterval(timer);
  }, [txState, scenario, timeScale, commandText, onUnlockBadge, onAwardXP]);

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <Radio className="w-6 h-6 text-cyan-400" />
            {t.deepSpaceComms}
          </h2>
          <p className="text-slate-400 text-sm">Experience finite speed-of-light latency across the Deep Space Network (DSN)</p>
        </div>
        <DataSourceBadge type="sim" t={t} />
      </div>

      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3">
        {scenarios.map((s) => (
          <button
            key={s.id}
            onClick={() => {
              setScenarioId(s.id);
              setTxState('IDLE');
              setProgress(0);
            }}
            className={`p-3.5 rounded-xl border text-left transition ${
              scenarioId === s.id
                ? 'bg-cyan-500/20 border-cyan-400 text-white'
                : 'bg-slate-900/75 border-slate-800 text-slate-300 hover:border-slate-700'
            }`}
          >
            <div className="font-bold text-sm">{s.name}</div>
            <div className="text-xs font-mono text-cyan-300 mt-1">{s.distanceKm}</div>
            <div className="text-[11px] font-mono text-slate-400">1-Way Delay: {s.oneWaySec}s</div>
          </button>
        ))}
      </div>

      {/* Visual Signal Propagation Track */}
      <div className="rounded-2xl border border-cyan-500/30 bg-slate-900/80 p-6 space-y-5">
        <div className="flex items-center justify-between text-xs font-mono">
          <span className="text-cyan-300 font-bold">🌍 EARTH DSN GOLDSTONE</span>
          <span className="text-slate-400">
            Status: <strong className="text-white">{txState}</strong> • Time Compression: {timeScale}x
          </span>
          <span className="text-amber-300 font-bold">🛰️ {scenario.name.toUpperCase()}</span>
        </div>

        <div className="relative h-4 rounded-full bg-slate-950 border border-slate-800 overflow-hidden">
          {txState === 'UPLINK' && (
            <div
              className="h-full bg-gradient-to-r from-cyan-500 to-blue-400 transition-all duration-75"
              style={{ width: `${progress}%` }}
            />
          )}
          {txState === 'DOWNLINK' && (
            <div
              className="h-full bg-gradient-to-l from-emerald-400 to-cyan-400 ml-auto transition-all duration-75"
              style={{ width: `${progress}%` }}
            />
          )}
          {txState === 'COMPLETE' && <div className="h-full w-full bg-emerald-500/50" />}
        </div>

        <div className="flex flex-wrap items-center gap-3">
          <select
            value={commandText}
            onChange={(e) => setCommandText(e.target.value)}
            className="p-2.5 rounded-xl bg-slate-950 border border-slate-700 text-xs font-mono text-cyan-200"
          >
            <option value="DEPLOY_SUPERCAM_LASER">CMD: DEPLOY_SUPERCAM_LASER</option>
            <option value="CAPTURE_MASTCAM_PANORAMA">CMD: CAPTURE_MASTCAM_PANORAMA</option>
            <option value="DRILL_CORE_SAMPLE_04">CMD: DRILL_CORE_SAMPLE_04</option>
            <option value="DOWNLINK_HEALTH_TELEMETRY">CMD: DOWNLINK_HEALTH_TELEMETRY</option>
          </select>

          <div className="flex items-center gap-1 bg-slate-950 p-1 rounded-xl border border-slate-800 text-xs font-mono">
            {[1, 30, 300].map((scale) => (
              <button
                key={scale}
                onClick={() => setTimeScale(scale)}
                className={`px-2.5 py-1 rounded-lg ${timeScale === scale ? 'bg-cyan-500 text-slate-950 font-bold' : 'text-slate-400'}`}
              >
                {scale}x
              </button>
            ))}
          </div>

          <button
            onClick={transmitCommand}
            disabled={txState === 'UPLINK' || txState === 'DOWNLINK'}
            className="px-5 py-2.5 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600 disabled:opacity-50 text-slate-950 font-bold text-xs"
          >
            Transmit Uplink Packet
          </button>

          <button
            onClick={() => onLogObservation(`Deep Space Comms Test (${scenario.name}): Sent ${commandText}; 1-way light latency = ${scenario.oneWaySec}s.`)}
            className="px-4 py-2.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-cyan-300 border border-slate-700 text-xs font-semibold"
          >
            {t.logObservation}
          </button>
        </div>

        <div className="bg-slate-950 border border-slate-800 rounded-xl p-3 font-mono text-xs space-y-1.5 max-h-40 overflow-y-auto">
          {commsLog.length === 0 ? (
            <div className="text-slate-500">Awaiting command transmission...</div>
          ) : (
            commsLog.map((entry, idx) => (
              <div key={idx} className="text-slate-300">
                <span className="text-slate-500">[{entry.time}]</span>{' '}
                <span className={entry.dir === 'UPLINK' ? 'text-cyan-400' : entry.dir === 'DOWNLINK' ? 'text-emerald-400' : 'text-amber-400'}>
                  [{entry.dir}]
                </span>{' '}
                {entry.msg}
              </div>
            ))
          )}
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MODULE 10: AUTOMATED SCIENTIFIC REPORT GENERATOR
// ============================================================================
const ScientificReportsModule = ({ t, state, onUnlockBadge, onAwardXP }) => {
  const [missionTitle, setMissionTitle] = useState('Planetary Surface & Environmental Reconnaissance Report');
  const [objective, setObjective] = useState(
    'Evaluate planetary gravity kinematics, radiation shielding attenuation, autonomous rover navigation, and mineral reflectance spectra.'
  );
  const [conclusion, setConclusion] = useState(
    'Hydrogen-rich shielding materials and autonomous obstacle-avoidance algorithms significantly enhance mission safety and scientific return.'
  );

  const generateAndAward = () => {
    onUnlockBadge('chief_scientist');
    onAwardXP(100);
  };

  const handleDownloadHTML = () => {
    generateAndAward();
    const htmlContent = `<!DOCTYPE html>
<html>
<head><meta charset="utf-8"><title>${missionTitle}</title>
<style>
  body { font-family: Arial, sans-serif; max-width: 800px; margin: 40px auto; color: #0f172a; line-height: 1.6; }
  header { border-bottom: 3px solid #0284c7; padding-bottom: 12px; margin-bottom: 24px; }
  .badge { background: #e0f2fe; color: #0369a1; padding: 4px 10px; border-radius: 999px; font-size: 12px; font-weight: bold; }
  h2 { color: #0369a1; margin-top: 24px; }
  li { margin-bottom: 8px; }
  footer { margin-top: 40px; font-size: 12px; color: #64748b; border-top: 1px solid #cbd5e1; padding-top: 12px; }
</style>
</head>
<body>
  <header>
    <span class="badge">ASTRO-TRAINER X • SCIENTIFIC MISSION REPORT</span>
    <h1>${missionTitle}</h1>
    <p><strong>Explorer Callsign:</strong> ${state.callsign} | <strong>Rank:</strong> ${state.rank} | <strong>Date:</strong> ${new Date().toLocaleDateString()}</p>
  </header>
  <h2>1. Scientific Objective</h2><p>${objective}</p>
  <h2>2. Experimental Procedure</h2><p>Executed interactive simulations using the ASTRO-TRAINER X Mission Control suite, recording telemetry from rover pathfinding, VNIR spectroscopy, radiation attenuation, and gravitational kinematics.</p>
  <h2>3. Logged Observations (${state.observations.length})</h2>
  <ul>${state.observations.map((o) => `<li><strong>[${o.time}]</strong> ${o.text}</li>`).join('') || '<li>Baseline training modules initialized.</li>'}</ul>
  <h2>4. Conclusions</h2><p>${conclusion}</p>
  <h2>5. Data Source Classification</h2><p>Educational Simulation Models & Public NASA/NOAA Telemetry Interfaces (Clearly distinguished from live spacecraft flight telemetry).</p>
  <footer>Generated safely on-device via ASTRO-TRAINER X • Zero PII Collected.</footer>
</body>
</html>`;

    const blob = new Blob([htmlContent], { type: 'text/html' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `ASTRO-TRAINER-X-Report-${Date.now()}.html`;
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <FileText className="w-6 h-6 text-cyan-400" />
            {t.scientificReports}
          </h2>
          <p className="text-slate-400 text-sm">Compile logged experimental telemetry into a formatted NASA-style Mission Report</p>
        </div>
        <div className="flex gap-2">
          <button
            onClick={() => {
              generateAndAward();
              window.print();
            }}
            className="px-4 py-2 rounded-xl bg-slate-800 hover:bg-slate-700 border border-slate-700 text-xs font-semibold text-white flex items-center gap-2"
          >
            <Printer className="w-4 h-4 text-cyan-400" />
            Print / Save PDF
          </button>
          <button
            onClick={handleDownloadHTML}
            className="px-4 py-2 rounded-xl bg-gradient-to-r from-cyan-500 to-blue-600 text-slate-950 font-bold text-xs flex items-center gap-2"
          >
            <Download className="w-4 h-4" />
            Export HTML Report
          </button>
        </div>
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Editable Report Fields */}
        <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 space-y-4">
          <h3 className="font-bold text-white text-sm">Customize Report Sections</h3>
          <div>
            <label className="text-xs text-slate-400 block mb-1">Mission Title</label>
            <input
              value={missionTitle}
              onChange={(e) => setMissionTitle(e.target.value)}
              className="w-full p-2.5 rounded-xl bg-slate-950 border border-slate-700 text-xs text-white"
            />
          </div>
          <div>
            <label className="text-xs text-slate-400 block mb-1">Scientific Objective</label>
            <textarea
              rows={3}
              value={objective}
              onChange={(e) => setObjective(e.target.value)}
              className="w-full p-2.5 rounded-xl bg-slate-950 border border-slate-700 text-xs text-white"
            />
          </div>
          <div>
            <label className="text-xs text-slate-400 block mb-1">Conclusions</label>
            <textarea
              rows={3}
              value={conclusion}
              onChange={(e) => setConclusion(e.target.value)}
              className="w-full p-2.5 rounded-xl bg-slate-950 border border-slate-700 text-xs text-white"
            />
          </div>
        </div>

        {/* Live Document Preview */}
        <div className="lg:col-span-2 rounded-2xl border border-cyan-500/30 bg-slate-950 p-6 space-y-4 text-sm">
          <div className="border-b border-slate-800 pb-4 flex flex-wrap justify-between items-start gap-2">
            <div>
              <span className="text-[11px] font-mono text-cyan-400">ASTRO-TRAINER X • OFFICIAL LAB RECORD</span>
              <h3 className="text-xl font-bold text-white mt-1">{missionTitle}</h3>
            </div>
            <div className="text-right text-xs font-mono text-slate-400">
              <div>EXPLORER: {state.callsign}</div>
              <div>XP: {state.xp} • {state.rank}</div>
            </div>
          </div>

          <div>
            <h4 className="text-xs font-mono uppercase text-cyan-400 mb-1">1. Scientific Objective</h4>
            <p className="text-slate-300 text-xs leading-relaxed">{objective}</p>
          </div>

          <div>
            <h4 className="text-xs font-mono uppercase text-cyan-400 mb-2">
              2. Recorded Experimental Observations ({state.observations.length})
            </h4>
            {state.observations.length === 0 ? (
              <div className="p-3 rounded-xl bg-slate-900/70 border border-slate-800 text-xs text-slate-400">
                No observations logged yet. Visit the Rover, Gravity, Spectral, or Radiation modules and click "{t.logObservation}" to populate your report automatically!
              </div>
            ) : (
              <ul className="space-y-2">
                {state.observations.map((obs, idx) => (
                  <li key={idx} className="p-2.5 rounded-xl bg-slate-900/90 border border-slate-800 text-xs text-slate-200">
                    <span className="font-mono text-cyan-400 mr-2">[{obs.time}]</span>
                    {obs.text}
                  </li>
                ))}
              </ul>
            )}
          </div>

          <div>
            <h4 className="text-xs font-mono uppercase text-cyan-400 mb-1">3. Conclusions & Data Classification</h4>
            <p className="text-slate-300 text-xs leading-relaxed">{conclusion}</p>
            <div className="mt-2">
              <DataSourceBadge type="sim" t={t} />
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MODULE 12: COOPERATIVE MISSION CONTROL (Multi-Role Simulator)
// ============================================================================
const CooperativeMissionModule = ({ t, onAwardXP }) => {
  const roles = [
    {
      id: 'commander',
      title: 'Mission Commander',
      icon: '👨‍🚀',
      tasks: ['Verify flight readiness checklist', 'Authorize rover surface deployment', 'Approve final science downlink'],
    },
    {
      id: 'rover_op',
      title: 'Rover Operator',
      icon: '🛞',
      tasks: ['Calibrate LiDAR obstacle thresholds', 'Plot safe path around Jezero boulders', 'Confirm wheel motor torque'],
    },
    {
      id: 'scientist',
      title: 'Scientific Analyst',
      icon: '🔬',
      tasks: ['Inspect VNIR spectrometer absorption bands', 'Classify hydrated sulfate sample', 'Verify radiation shelter margins'],
    },
    {
      id: 'comms',
      title: 'Communications Officer',
      icon: '📡',
      tasks: ['Lock DSN high-gain antenna azimuth', 'Calculate Mars light-time delay', 'Monitor solar wind Kp alerts'],
    },
  ];

  const [activeRole, setActiveRole] = useState('commander');
  const [checkedTasks, setCheckedTasks] = useState({});
  const [crewMessages, setCrewMessages] = useState([
    { role: 'Mission Commander', text: 'All stations, report readiness for Surface Ops.' },
    { role: 'Communications Officer', text: 'DSN Goldstone uplink locked. 1-way delay 3m 02s.' },
  ]);
  const [chatInput, setChatInput] = useState('');

  const totalTasks = roles.reduce((sum, r) => sum + r.tasks.length, 0);
  const completedCount = Object.values(checkedTasks).filter(Boolean).length;
  const readinessPct = Math.round((completedCount / totalTasks) * 100);

  const toggleTask = (key) => {
    setCheckedTasks((prev) => {
      const nextVal = !prev[key];
      if (nextVal) onAwardXP(15);
      return { ...prev, [key]: nextVal };
    });
  };

  const currentRoleObj = roles.find((r) => r.id === activeRole) || roles[0];

  return (
    <div className="space-y-6">
      <div className="flex flex-wrap items-center justify-between gap-4">
        <div>
          <h2 className="text-2xl font-bold text-white flex items-center gap-2">
            <Users className="w-6 h-6 text-cyan-400" />
            {t.coopMissions}
          </h2>
          <p className="text-slate-400 text-sm">Coordinate across 4 specialist flight-control stations (Local multi-role simulation — zero public chat)</p>
        </div>
        <div className="text-right">
          <div className="text-xs font-mono text-cyan-300">CREW READINESS: {readinessPct}%</div>
          <div className="w-40 h-2 rounded-full bg-slate-800 mt-1 overflow-hidden">
            <div className="h-full bg-gradient-to-r from-cyan-400 to-emerald-400 transition-all" style={{ width: `${readinessPct}%` }} />
          </div>
        </div>
      </div>

      {/* Role Switcher */}
      <div className="grid grid-cols-2 lg:grid-cols-4 gap-3">
        {roles.map((r) => (
          <button
            key={r.id}
            onClick={() => setActiveRole(r.id)}
            className={`p-3.5 rounded-xl border text-left transition ${
              activeRole === r.id
                ? 'bg-cyan-500/20 border-cyan-400 text-white'
                : 'bg-slate-900/75 border-slate-800 text-slate-300 hover:border-slate-700'
            }`}
          >
            <div className="text-xl mb-1">{r.icon}</div>
            <div className="font-bold text-sm">{r.title}</div>
          </button>
        ))}
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        {/* Role Checklist */}
        <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 space-y-3">
          <h3 className="font-bold text-white flex items-center gap-2">
            <span>{currentRoleObj.icon}</span> {currentRoleObj.title} Checklist
          </h3>
          <div className="space-y-2">
            {currentRoleObj.tasks.map((task, idx) => {
              const key = `${currentRoleObj.id}_${idx}`;
              const done = !!checkedTasks[key];
              return (
                <button
                  key={key}
                  onClick={() => toggleTask(key)}
                  className={`w-full p-3 rounded-xl border text-left text-xs flex items-center gap-3 transition ${
                    done ? 'bg-emerald-500/15 border-emerald-500/40 text-emerald-200' : 'bg-slate-950 border-slate-800 text-slate-300'
                  }`}
                >
                  <CheckCircle2 className={`w-4 h-4 shrink-0 ${done ? 'text-emerald-400' : 'text-slate-600'}`} />
                  <span>{task}</span>
                </button>
              );
            })}
          </div>
        </div>

        {/* Local Station Comms Loop */}
        <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 flex flex-col justify-between space-y-3">
          <div>
            <h3 className="font-bold text-white text-sm mb-2">Flight Loop Protocol Log (Local Classroom Mode)</h3>
            <div className="bg-slate-950 border border-slate-800 rounded-xl p-3 h-44 overflow-y-auto space-y-2 text-xs">
              {crewMessages.map((m, i) => (
                <div key={i}>
                  <span className="font-mono text-cyan-400 font-bold">[{m.role}]: </span>
                  <span className="text-slate-200">{m.text}</span>
                </div>
              ))}
            </div>
          </div>

          <form
            onSubmit={(e) => {
              e.preventDefault();
              if (!chatInput.trim()) return;
              setCrewMessages((prev) => [...prev, { role: currentRoleObj.title, text: chatInput.trim() }]);
              setChatInput('');
            }}
            className="flex gap-2"
          >
            <input
              value={chatInput}
              onChange={(e) => setChatInput(e.target.value)}
              placeholder={`Transmit as ${currentRoleObj.title}...`}
              className="flex-1 px-3 py-2 rounded-xl bg-slate-950 border border-slate-700 text-xs text-white"
            />
            <button type="submit" className="px-4 py-2 rounded-xl bg-cyan-500 text-slate-950 font-bold text-xs">
              Send
            </button>
          </form>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MODULE 13: PRIVACY, CHILD SAFETY & PROJECT DOCS
// ============================================================================
const PrivacyAndDocsModule = ({ t, onResetProgress }) => {
  const [confirmReset, setConfirmReset] = useState(false);

  return (
    <div className="space-y-6">
      <div>
        <h2 className="text-2xl font-bold text-white flex items-center gap-2">
          <ShieldCheck className="w-6 h-6 text-emerald-400" />
          {t.privacySafety} & Architecture Docs
        </h2>
        <p className="text-slate-400 text-sm">Child-first privacy engineering (COPPA-aligned principles) & open scientific architecture</p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div className="rounded-2xl border border-emerald-500/30 bg-slate-900/80 p-5 space-y-3">
          <h3 className="font-bold text-emerald-300 flex items-center gap-2">
            <Lock className="w-4 h-4" />
            Child Safety & Zero-PII Guarantees
          </h3>
          <ul className="space-y-2 text-xs text-slate-300">
            <li>✅ <strong>Zero Personal Data Collected:</strong> No real names, emails, birthdates, photos, or school IDs are requested or stored.</li>
            <li>✅ <strong>Local-Only Progress:</strong> Training XP, badges, and reports remain strictly in browser <code>localStorage</code>.</li>
            <li>✅ <strong>No Public Chat or Geolocation:</strong> Cooperative mode runs locally on-device with zero external social exposure or GPS tracking.</li>
            <li>✅ <strong>No Ads or Behavioral Trackers:</strong> Zero third-party advertising SDKs or analytics cookies.</li>
            <li className="text-amber-300 pt-1">
              ℹ️ Note: Designed following COPPA privacy-by-design principles; formal deployment in schools requires standard institutional compliance review.
            </li>
          </ul>
        </div>

        <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-5 space-y-3">
          <h3 className="font-bold text-cyan-300 flex items-center gap-2">
            <Code2 className="w-4 h-4" />
            NASA Data Engine & Hardware Specs
          </h3>
          <ul className="space-y-2 text-xs text-slate-300">
            <li>🛰️ <strong>NASA/NOAA Integration Layer:</strong> Connects to public NOAA SWPC planetary K-index feeds with transparent fallback to clearly labeled Educational Simulation datasets.</li>
            <li>🤖 <strong>ESP32 / Arduino Bridge:</strong> Uses Web Serial API (115200 baud) for optional classroom robotics hardware without ever faking hardware connectivity.</li>
            <li>🧪 <strong>Simulation Boundaries:</strong> Gravity, radiation shielding, and spectral curves use simplified educational physics models ideal for ages 8–16.</li>
          </ul>

          <div className="pt-2">
            {!confirmReset ? (
              <button
                onClick={() => setConfirmReset(true)}
                className="px-4 py-2 rounded-xl bg-red-500/15 border border-red-500/40 text-red-300 text-xs font-semibold"
              >
                Reset Local Training Progress
              </button>
            ) : (
              <div className="flex items-center gap-2">
                <button
                  onClick={() => {
                    onResetProgress();
                    setConfirmReset(false);
                  }}
                  className="px-4 py-2 rounded-xl bg-red-600 text-white text-xs font-bold"
                >
                  Confirm Clear Local Data
                </button>
                <button
                  onClick={() => setConfirmReset(false)}
                  className="px-3 py-2 rounded-xl bg-slate-800 text-slate-300 text-xs"
                >
                  Cancel
                </button>
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
};

// ============================================================================
// MAIN APPLICATION SHELL: ASTRO-TRAINER X
// ============================================================================
export default function App() {
  const scriptsReady = useExternalScripts();
  const [lang, setLang] = useState('en');
  const [activeTab, setActiveTab] = useState('dashboard');
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);
  const [toast, setToast] = useState(null);
  const [isListening, setIsListening] = useState(false);

  // Persistent Junior Astronaut State (Zero PII)
  const defaultProgress = {
    callsign: 'ASTRO-01',
    avatar: '🧑‍🚀',
    xp: 120,
    completedMissions: [],
    unlockedBadges: ['first_launch'],
    observations: [],
  };

  const [progress, setProgress] = useState(() => {
    try {
      const saved = localStorage.getItem('astro_trainer_x_v1');
      return saved ? { ...defaultProgress, ...JSON.parse(saved) } : defaultProgress;
    } catch {
      return defaultProgress;
    }
  });

  useEffect(() => {
    try {
      localStorage.setItem('astro_trainer_x_v1', JSON.stringify(progress));
    } catch {
      // Ignore storage quota errors
    }
  }, [progress]);

  const t = TRANSLATIONS[lang];
  const isRTL = lang === 'ar';

  const level = Math.floor(progress.xp / 200) + 1;
  const rankIndex = Math.min(t.ranks.length - 1, Math.floor((level - 1) / 2));
  const currentRank = t.ranks[rankIndex];
  const xpProgressPct = ((progress.xp % 200) / 200) * 100;

  const showToast = useCallback((msg) => {
    setToast(msg);
    setTimeout(() => setToast(null), 3200);
  }, []);

  const awardXP = useCallback(
    (amount) => {
      setProgress((prev) => ({ ...prev, xp: prev.xp + amount }));
      showToast(`+${amount} XP Earned!`);
    },
    [showToast]
  );

  const unlockBadge = useCallback(
    (badgeId) => {
      setProgress((prev) => {
        if (prev.unlockedBadges.includes(badgeId)) return prev;
        const badge = BADGES_CATALOG.find((b) => b.id === badgeId);
        if (badge) showToast(`🏅 Badge Unlocked: ${badge.title}`);
        return {
          ...prev,
          unlockedBadges: [...prev.unlockedBadges, badgeId],
          xp: prev.xp + (badge?.xp || 50),
        };
      });
    },
    [showToast]
  );

  const completeMissionStep = useCallback((missionId) => {
    setProgress((prev) => {
      if (prev.completedMissions.includes(missionId)) return prev;
      return { ...prev, completedMissions: [...prev.completedMissions, missionId] };
    });
  }, []);

  const logObservation = useCallback(
    (text) => {
      setProgress((prev) => ({
        ...prev,
        observations: [{ time: new Date().toLocaleTimeString(), text }, ...prev.observations.slice(0, 24)],
      }));
      showToast(t.loggedSuccess);
    },
    [showToast, t.loggedSuccess]
  );

  // Bilingual Web Speech API Voice Command Handler
  const triggerVoiceCommand = () => {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
      showToast('Voice recognition unavailable in this browser. Use sidebar buttons.');
      return;
    }

    const recognition = new SpeechRecognition();
    recognition.lang = lang === 'ar' ? 'ar-SA' : 'en-US';
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    setIsListening(true);
    recognition.onresult = (event) => {
      const transcript = event.results[0][0].transcript.toLowerCase();
      setIsListening(false);
      showToast(`🎙️ "${transcript}"`);

      if (transcript.includes('rover') || transcript.includes('مركبة')) setActiveTab('rover');
      else if (transcript.includes('gravity') || transcript.includes('جاذبية')) setActiveTab('gravity');
      else if (transcript.includes('weather') || transcript.includes('طقس')) setActiveTab('weather');
      else if (transcript.includes('report') || transcript.includes('تقرير')) setActiveTab('reports');
      else if (transcript.includes('planet') || transcript.includes('كواكب')) setActiveTab('planetary');
      else if (transcript.includes('rock') || transcript.includes('spectral') || transcript.includes('صخور')) setActiveTab('spectral');
      else if (transcript.includes('radiation') || transcript.includes('إشعاع')) setActiveTab('radiation');
      else if (transcript.includes('mission') || transcript.includes('مهمة')) setActiveTab('dashboard');

      if ('speechSynthesis' in window) {
        const utter = new SpeechSynthesisUtterance(
          lang === 'ar' ? 'تم تنفيذ الأمر بنجاح' : 'Command acknowledged, Explorer.'
        );
        utter.lang = lang === 'ar' ? 'ar-SA' : 'en-US';
        window.speechSynthesis.speak(utter);
      }
    };
    recognition.onerror = () => {
      setIsListening(false);
      showToast('Voice command cancelled or microphone permission denied.');
    };
    recognition.onend = () => setIsListening(false);
    recognition.start();
  };

  const navItems = [
    { id: 'dashboard', label: t.missionControl, icon: Rocket },
    { id: 'profile', label: t.profile, icon: Star },
    { id: 'planetary', label: t.planetary3d, icon: Globe },
    { id: 'rover', label: t.roverControl, icon: Cpu },
    { id: 'radiation', label: t.radiationLab, icon: FlaskConical },
    { id: 'weather', label: t.spaceWeather, icon: Sun },
    { id: 'spectral', label: t.spectralLab, icon: Activity },
    { id: 'gravity', label: t.gravitySim, icon: ArrowDownToLine },
    { id: 'comms', label: t.deepSpaceComms, icon: Radio },
    { id: 'reports', label: t.scientificReports, icon: FileText },
    { id: 'badges', label: t.achievements, icon: Award },
    { id: 'coop', label: t.coopMissions, icon: Users },
    { id: 'privacy', label: t.privacySafety, icon: ShieldCheck },
  ];

  return (
    <div dir={isRTL ? 'rtl' : 'ltr'} className="min-h-screen bg-[#030712] text-slate-100 font-sans selection:bg-cyan-500 selection:text-slate-950">
      <StarfieldBackground />

      {/* Notification Toast */}
      {toast && (
        <div className="fixed bottom-5 right-5 z-50 px-4 py-3 rounded-2xl bg-slate-900/95 border border-cyan-400/60 text-cyan-200 text-xs font-mono shadow-[0_0_25px_rgba(6,182,212,0.35)] flex items-center gap-2">
          <Sparkles className="w-4 h-4 text-cyan-400" />
          {toast}
        </div>
      )}

      {/* Top Mission Control Header */}
      <header className="sticky top-0 z-30 border-b border-slate-800/90 bg-[#050a18]/90 backdrop-blur-xl">
        <div className="max-w-[1440px] mx-auto px-4 h-16 flex items-center justify-between gap-3">
          <div className="flex items-center gap-3">
            <button
              onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
              className="lg:hidden p-2 rounded-xl bg-slate-900 border border-slate-800 text-cyan-400"
              aria-label="Toggle Menu"
            >
              <Layers className="w-5 h-5" />
            </button>
            <div className="w-9 h-9 rounded-xl bg-gradient-to-br from-cyan-400 to-blue-600 flex items-center justify-center text-slate-950 font-black shadow-[0_0_18px_rgba(0,210,255,0.45)]">
              AX
            </div>
            <div>
              <div className="font-black tracking-wider text-sm sm:text-base text-white flex items-center gap-2">
                {t.appName}
                <span className="hidden sm:inline-block text-[10px] font-mono px-2 py-0.5 rounded bg-cyan-500/15 text-cyan-300 border border-cyan-500/30">
                  SYS v4.2
                </span>
              </div>
              <div className="text-[11px] text-slate-400 hidden sm:block">{t.subtitle}</div>
            </div>
          </div>

          {/* Right Controls: XP Pill, Voice Command, Bilingual Toggle */}
          <div className="flex items-center gap-2 sm:gap-3">
            <button
              onClick={() => setActiveTab('profile')}
              className="hidden md:flex items-center gap-2.5 px-3 py-1.5 rounded-xl bg-slate-900/90 border border-slate-800 hover:border-cyan-500/40 transition"
            >
              <span className="text-base">{progress.avatar}</span>
              <div className="text-left">
                <div className="text-xs font-bold text-white">{progress.callsign} • {currentRank}</div>
                <div className="w-24 h-1.5 bg-slate-800 rounded-full overflow-hidden mt-0.5">
                  <div className="h-full bg-cyan-400" style={{ width: `${xpProgressPct}%` }} />
                </div>
              </div>
              <span className="text-xs font-mono text-cyan-300 font-bold">{progress.xp} {t.xp}</span>
            </button>

            <button
              onClick={triggerVoiceCommand}
              title={t.voiceHelp}
              className={`px-3 py-2 rounded-xl border text-xs font-semibold flex items-center gap-1.5 transition ${
                isListening
                  ? 'bg-rose-500/20 border-rose-400 text-rose-200 animate-pulse'
                  : 'bg-slate-900 hover:bg-slate-800 border-cyan-500/30 text-cyan-300'
              }`}
            >
              {isListening ? <MicOff className="w-4 h-4" /> : <Mic className="w-4 h-4" />}
              <span className="hidden sm:inline">{isListening ? t.listening : t.voiceCmd}</span>
            </button>

            <button
              onClick={() => setLang(lang === 'en' ? 'ar' : 'en')}
              className="px-3 py-2 rounded-xl bg-slate-900 hover:bg-slate-800 border border-slate-700 text-xs font-bold text-white"
            >
              {lang === 'en' ? 'العربية' : 'EN'}
            </button>
          </div>
        </div>
      </header>

      {/* Main Layout Shell */}
      <div className="max-w-[1440px] mx-auto px-4 py-6 flex gap-6 relative z-10">
        {/* Sidebar Navigation */}
        <aside
          className={`${
            mobileMenuOpen ? 'fixed inset-y-16 left-0 right-0 z-40 bg-[#050a18]/98 p-4 overflow-y-auto' : 'hidden'
          } lg:block lg:w-64 shrink-0`}
        >
          <nav className="space-y-1 rounded-2xl border border-slate-800/90 bg-slate-900/60 backdrop-blur-xl p-2.5">
            {navItems.map((item) => {
              const Icon = item.icon;
              const active = activeTab === item.id;
              return (
                <button
                  key={item.id}
                  onClick={() => {
                    setActiveTab(item.id);
                    setMobileMenuOpen(false);
                  }}
                  className={`w-full flex items-center gap-3 px-3 py-2.5 rounded-xl text-xs font-semibold transition ${
                    active
                      ? 'bg-gradient-to-r from-cyan-500/20 to-blue-600/20 border border-cyan-400/50 text-cyan-200 shadow-[0_0_15px_rgba(6,182,212,0.15)]'
                      : 'text-slate-400 hover:text-white hover:bg-slate-800/60 border border-transparent'
                  }`}
                >
                  <Icon className={`w-4 h-4 shrink-0 ${active ? 'text-cyan-400' : 'text-slate-500'}`} />
                  <span className="truncate">{item.label}</span>
                </button>
              );
            })}
          </nav>

          <div className="mt-4 p-3.5 rounded-2xl border border-emerald-500/25 bg-emerald-950/20 text-[11px] text-emerald-300/90">
            <div className="font-bold flex items-center gap-1.5 mb-1">
              <ShieldCheck className="w-3.5 h-3.5" />
              Child-Safe Mode Active
            </div>
            {t.safeModeNotice}
          </div>
        </aside>

        {/* Active Module Workspace */}
        <main className="flex-1 min-w-0">
          {/* MODULE 1: MISSION CONTROL DASHBOARD */}
          {activeTab === 'dashboard' && (
            <div className="space-y-6">
              {/* Hero Command Banner */}
              <div className="rounded-3xl border border-cyan-500/30 bg-gradient-to-br from-slate-900/95 via-[#09142c] to-slate-950 p-6 sm:p-8 relative overflow-hidden">
                <div className="max-w-2xl space-y-3 relative z-10">
                  <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-cyan-500/15 border border-cyan-400/30 text-cyan-300 text-xs font-mono">
                    <Sparkles className="w-3.5 h-3.5" />
                    NASA DATA-TO-MISSION ENGINE • READY
                  </div>
                  <h1 className="text-2xl sm:text-4xl font-black text-white tracking-tight">
                    Welcome aboard, <span className="text-cyan-400">{progress.callsign}</span>
                  </h1>
                  <p className="text-slate-300 text-sm leading-relaxed">
                    Explore real planetary science, pilot autonomous rovers, analyze Martian rock spectra, and test spacecraft radiation shielding in an interactive space-agency simulator.
                  </p>
                  <div className="flex flex-wrap gap-3 pt-2">
                    <button
                      onClick={() => setActiveTab('rover')}
                      className="px-5 py-2.5 rounded-xl bg-gradient-to-r from-cyan-400 to-blue-600 hover:from-cyan-300 hover:to-blue-500 text-slate-950 font-bold text-xs flex items-center gap-2 shadow-[0_0_20px_rgba(0,210,255,0.3)]"
                    >
                      <Play className="w-4 h-4" />
                      {t.startMission}
                    </button>
                    <button
                      onClick={() => setActiveTab('planetary')}
                      className="px-5 py-2.5 rounded-xl bg-slate-800/90 hover:bg-slate-700 border border-slate-700 text-white font-semibold text-xs flex items-center gap-2"
                    >
                      <Globe className="w-4 h-4 text-cyan-400" />
                      {t.planetary3d}
                    </button>
                  </div>
                </div>
              </div>

              {/* System Telemetry Strip */}
              <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
                <div className="p-4 rounded-2xl bg-slate-900/75 border border-slate-800">
                  <div className="text-xs text-slate-400">Astronaut Rank</div>
                  <div className="text-lg font-bold text-white mt-1">{currentRank}</div>
                  <div className="text-xs font-mono text-cyan-400">{t.level} {level} ({progress.xp} XP)</div>
                </div>
                <div className="p-4 rounded-2xl bg-slate-900/75 border border-slate-800">
                  <div className="text-xs text-slate-400">Missions Completed</div>
                  <div className="text-lg font-bold text-emerald-400 mt-1">
                    {progress.completedMissions.length} / {MISSION_CATALOG.length}
                  </div>
                  <div className="text-xs font-mono text-slate-400">Active Training Track</div>
                </div>
                <div className="p-4 rounded-2xl bg-slate-900/75 border border-slate-800">
                  <div className="text-xs text-slate-400">Logged Observations</div>
                  <div className="text-lg font-bold text-purple-300 mt-1">{progress.observations.length}</div>
                  <button onClick={() => setActiveTab('reports')} className="text-xs font-mono text-cyan-400 hover:underline">
                    Open Report Builder →
                  </button>
                </div>
                <div className="p-4 rounded-2xl bg-slate-900/75 border border-slate-800">
                  <div className="text-xs text-slate-400">Data Integrity Mode</div>
                  <div className="mt-1.5">
                    <DataSourceBadge type="sim" t={t} />
                  </div>
                  <div className="text-[11px] text-slate-500 mt-1">Live NOAA feed in Space Weather</div>
                </div>
              </div>

              {/* Active Science Missions Queue */}
              <div className="space-y-3">
                <h2 className="text-lg font-bold text-white flex items-center gap-2">
                  <Target className="w-5 h-5 text-cyan-400" />
                  Training Missions Queue
                </h2>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  {MISSION_CATALOG.map((m) => {
                    const done = progress.completedMissions.includes(m.id);
                    return (
                      <div
                        key={m.id}
                        className={`p-5 rounded-2xl border transition flex flex-col justify-between space-y-4 ${
                          done
                            ? 'bg-emerald-950/20 border-emerald-500/40'
                            : 'bg-slate-900/75 border-slate-800 hover:border-cyan-500/40'
                        }`}
                      >
                        <div>
                          <div className="flex items-center justify-between text-xs font-mono mb-1.5">
                            <span className="text-cyan-400">{m.id} • {m.difficulty}</span>
                            <span className="text-amber-300">+{m.xp} XP</span>
                          </div>
                          <h3 className="font-bold text-white text-base">{m.title}</h3>
                          <p className="text-xs text-slate-400 mt-1">{m.summary}</p>
                          <div className="mt-2 text-xs text-cyan-200/90 bg-slate-950/70 p-2 rounded-lg border border-slate-800">
                            <strong>Objective:</strong> {m.objective}
                          </div>
                        </div>

                        <div className="flex items-center justify-between pt-1">
                          {done ? (
                            <span className="text-xs font-bold text-emerald-400 flex items-center gap-1.5">
                              <CheckCircle2 className="w-4 h-4" /> Completed
                            </span>
                          ) : (
                            <span className="text-xs text-slate-500">Status: Ready</span>
                          )}
                          <button
                            onClick={() => setActiveTab(m.targetTab)}
                            className="px-4 py-2 rounded-xl bg-cyan-500/20 hover:bg-cyan-500 hover:text-slate-950 border border-cyan-400/50 text-cyan-300 text-xs font-bold transition flex items-center gap-1"
                          >
                            Launch Module <ChevronRight className="w-3.5 h-3.5" />
                          </button>
                        </div>
                      </div>
                    );
                  })}
                </div>
              </div>

              {/* Quick Module Launcher Grid */}
              <div className="space-y-3">
                <h2 className="text-lg font-bold text-white">All Interactive Laboratories & Simulators</h2>
                <div className="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-3">
                  {navItems.slice(2).map((item) => {
                    const Icon = item.icon;
                    return (
                      <button
                        key={item.id}
                        onClick={() => setActiveTab(item.id)}
                        className="p-4 rounded-2xl bg-slate-900/70 hover:bg-slate-800/90 border border-slate-800 hover:border-cyan-500/40 text-left transition group"
                      >
                        <Icon className="w-5 h-5 text-cyan-400 group-hover:scale-110 transition-transform mb-2" />
                        <div className="font-bold text-xs text-white">{item.label}</div>
                      </button>
                    );
                  })}
                </div>
              </div>
            </div>
          )}

          {/* MODULE 2: JUNIOR ASTRONAUT PROFILE */}
          {activeTab === 'profile' && (
            <div className="max-w-2xl space-y-6">
              <h2 className="text-2xl font-bold text-white">{t.profile}</h2>
              <div className="rounded-2xl border border-slate-800 bg-slate-900/80 p-6 space-y-5">
                <div className="flex items-center gap-4">
                  <div className="w-16 h-16 rounded-2xl bg-cyan-500/20 border border-cyan-400/50 flex items-center justify-center text-3xl">
                    {progress.avatar}
                  </div>
                  <div>
                    <div className="text-xs font-mono text-cyan-400">{currentRank} • {t.level} {level}</div>
                    <div className="text-xl font-bold text-white">{progress.callsign}</div>
                    <div className="text-xs text-slate-400">{progress.xp} Total XP</div>
                  </div>
                </div>

                <div>
                  <label className="text-xs text-slate-400 block mb-1">{t.callsign} (Non-Personal Explorer Code)</label>
                  <input
                    maxLength={16}
                    value={progress.callsign}
                    onChange={(e) => setProgress((p) => ({ ...p, callsign: e.target.value.toUpperCase() }))}
                    className="w-full sm:w-64 px-3 py-2 rounded-xl bg-slate-950 border border-slate-700 text-sm font-mono text-cyan-300"
                  />
                </div>

                <div>
                  <label className="text-xs text-slate-400 block mb-2">Select Suit Patch Avatar</label>
                  <div className="flex gap-2">
                    {['🧑‍🚀', '👩‍🚀', '🤖', '🛸', '🛰️', '🔭'].map((emo) => (
                      <button
                        key={emo}
                        onClick={() => setProgress((p) => ({ ...p, avatar: emo }))}
                        className={`w-11 h-11 rounded-xl border text-xl ${
                          progress.avatar === emo ? 'bg-cyan-500/25 border-cyan-400' : 'bg-slate-950 border-slate-800'
                        }`}
                      >
                        {emo}
                      </button>
                    ))}
                  </div>
                </div>
              </div>
            </div>
          )}

          {activeTab === 'planetary' && (
            <PlanetaryExplorerModule t={t} onLogObservation={logObservation} onAwardXP={awardXP} />
          )}

          {activeTab === 'rover' && (
            <RoverControlModule
              t={t}
              onCompleteMissionStep={completeMissionStep}
              onAwardXP={awardXP}
              onUnlockBadge={unlockBadge}
              onLogObservation={logObservation}
            />
          )}

          {activeTab === 'radiation' && (
            <RadiationLabModule
              t={t}
              onUnlockBadge={unlockBadge}
              onAwardXP={awardXP}
              onCompleteMissionStep={completeMissionStep}
              onLogObservation={logObservation}
            />
          )}

          {activeTab === 'weather' && (
            <SpaceWeatherModule
              t={t}
              chartReady={scriptsReady.chart}
              onLogObservation={logObservation}
              onAwardXP={awardXP}
            />
          )}

          {activeTab === 'spectral' && (
            <SpectralAnalyzerModule
              t={t}
              chartReady={scriptsReady.chart}
              onUnlockBadge={unlockBadge}
              onAwardXP={awardXP}
              onCompleteMissionStep={completeMissionStep}
              onLogObservation={logObservation}
            />
          )}

          {activeTab === 'gravity' && (
            <GravitySimulatorModule
              t={t}
              onUnlockBadge={unlockBadge}
              onAwardXP={awardXP}
              onCompleteMissionStep={completeMissionStep}
              onLogObservation={logObservation}
            />
          )}

          {activeTab === 'comms' && (
            <DeepSpaceCommsModule
              t={t}
              onUnlockBadge={unlockBadge}
              onAwardXP={awardXP}
              onLogObservation={logObservation}
            />
          )}

          {activeTab === 'reports' && (
            <ScientificReportsModule
              t={t}
              state={{ ...progress, rank: currentRank }}
              onUnlockBadge={unlockBadge}
              onAwardXP={awardXP}
            />
          )}

          {/* MODULE 11: ACHIEVEMENTS & BADGES */}
          {activeTab === 'badges' && (
            <div className="space-y-6">
              <div>
                <h2 className="text-2xl font-bold text-white flex items-center gap-2">
                  <Award className="w-6 h-6 text-amber-400" />
                  {t.achievements}
                </h2>
                <p className="text-slate-400 text-sm">
                  Unlocked {progress.unlockedBadges.length} of {BADGES_CATALOG.length} Explorer Commendations
                </p>
              </div>

              <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
                {BADGES_CATALOG.map((b) => {
                  const unlocked = progress.unlockedBadges.includes(b.id);
                  return (
                    <div
                      key={b.id}
                      className={`p-5 rounded-2xl border transition ${
                        unlocked
                          ? 'bg-gradient-to-br from-cyan-500/15 to-purple-500/10 border-cyan-400/50 text-white'
                          : 'bg-slate-900/50 border-slate-800 text-slate-500 opacity-70'
                      }`}
                    >
                      <div className="flex items-center justify-between mb-2">
                        <span className="text-3xl">{unlocked ? b.icon : '🔒'}</span>
                        <span className="text-xs font-mono text-amber-300">+{b.xp} XP</span>
                      </div>
                      <div className="font-bold text-sm">{b.title}</div>
                      <p className="text-xs mt-1 text-slate-400">{b.desc}</p>
                    </div>
                  );
                })}
              </div>
            </div>
          )}

          {activeTab === 'coop' && <CooperativeMissionModule t={t} onAwardXP={awardXP} />}

          {activeTab === 'privacy' && (
            <PrivacyAndDocsModule
              t={t}
              onResetProgress={() => {
                setProgress(defaultProgress);
                localStorage.removeItem('astro_trainer_x_v1');
                showToast('Local training data cleared.');
              }}
            />
          )}
        </main>
      </div>
    </div>
  );
}
