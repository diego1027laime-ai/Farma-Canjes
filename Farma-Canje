import React, { useState, useEffect, useRef, useMemo } from 'react';

// ============================================================================
// 1. ESCUDO ANTIFALLOS (SAFE ERROR BOUNDARY) - CERO PANTALLAS EN NEGRO
// ============================================================================
class SafeErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, errorMsg: '' };
  }
  static getDerivedStateFromError(error) {
    return { hasError: true, errorMsg: error ? error.toString() : 'Error inesperado' };
  }
  componentDidCatch(error, info) {
    console.error('Error capturado en FarmaCanjes Pro:', error, info);
  }
  render() {
    if (this.state.hasError) {
      return (
        <div className="min-h-screen bg-slate-900 text-white flex flex-col items-center justify-center p-6 text-center font-sans">
          <div className="bg-emerald-950/80 border border-emerald-500/40 p-7 rounded-3xl max-w-md w-full shadow-2xl">
            <div className="text-4xl mb-3">🛡️</div>
            <h2 className="text-xl font-black text-emerald-400 mb-2">Recuperación Automática Activada</h2>
            <p className="text-xs text-slate-300 mb-5">{this.state.errorMsg}</p>
            <button
              onClick={() => this.setState({ hasError: false })}
              className="w-full py-3.5 bg-emerald-600 hover:bg-emerald-500 text-white font-black rounded-2xl shadow-lg transition mb-2"
            >
              Continuar en el Aplicativo
            </button>
            <button
              onClick={() => {
                localStorage.clear();
                window.location.reload();
              }}
              className="w-full py-2.5 bg-slate-800 hover:bg-slate-700 text-slate-300 font-bold text-xs rounded-xl transition"
            >
              Limpiar Caché y Reiniciar
            </button>
          </div>
        </div>
      );
    }
    return this.props.children;
  }
}

// ============================================================================
// 2. SISTEMA DE SONIDO SINTETIZADO (LAZY WEB AUDIO API)
// Solo se inicializa después de un clic del usuario para respetar al navegador
// ============================================================================
class SoundSystem {
  constructor() {
    this.ctx = null;
    this.enabled = true;
    this.volume = 0.6;
  }
  init() {
    if (!this.ctx && typeof window !== 'undefined') {
      const AudioCtx = window.AudioContext || window.webkitAudioContext;
      if (AudioCtx) this.ctx = new AudioCtx();
    }
    if (this.ctx && this.ctx.state === 'suspended') {
      this.ctx.resume().catch(() => {});
    }
  }
  playTone(freq, type = 'sine', duration = 0.12, delay = 0) {
    if (!this.enabled) return;
    try {
      this.init();
      if (!this.ctx) return;
      const now = this.ctx.currentTime + delay;
      const osc = this.ctx.createOscillator();
      const gain = this.ctx.createGain();
      osc.type = type;
      osc.frequency.setValueAtTime(freq, now);
      gain.gain.setValueAtTime(Math.max(0.001, this.volume * 0.25), now);
      gain.gain.exponentialRampToValueAtTime(0.001, now + duration);
      osc.connect(gain);
      gain.connect(this.ctx.destination);
      osc.start(now);
      osc.stop(now + duration);
    } catch (e) {}
  }
  playSelect() { this.playTone(640, 'sine', 0.08); }
  playAdd() { this.playTone(523.25, 'triangle', 0.1, 0); this.playTone(659.25, 'triangle', 0.15, 0.08); }
  playDelete() { this.playTone(300, 'sine', 0.11, 0); this.playTone(210, 'sine', 0.15, 0.06); }
  playSave() { this.playTone(440, 'sine', 0.1, 0); this.playTone(554.37, 'sine', 0.1, 0.08); this.playTone(659.25, 'sine', 0.2, 0.16); }
  playPDF() { this.playTone(587.33, 'sine', 0.1, 0); this.playTone(880, 'sine', 0.25, 0.1); }
  playError() { this.playTone(200, 'sawtooth', 0.16, 0); this.playTone(170, 'sawtooth', 0.22, 0.12); }
  playExcelSuccess() {
    [523.25, 659.25, 783.99, 1046.5].forEach((f, i) => this.playTone(f, 'sine', 0.12, i * 0.07));
  }
}

const soundFX = new SoundSystem();

// ============================================================================
// 3. BASE DE DATOS FARMACÉUTICA INICIAL (LISTA PARA PROBAR AL INSTANTE)
// ============================================================================
const INITIAL_CATALOG = [
  { codigo: '104521', descripcion: 'PANADOL FORTE 500MG/65MG CAJA X 100 TAB', laboratorio: 'GSK CONSUMER', presentacion: 'CAJA X 100 TAB', tipo: 'MEDICAMENTO', vencimiento: '2026-11-30', ean: '7750104521001' },
  { codigo: '208914', descripcion: 'AMOXICILINA 500MG GENFAR CAJA X 50 CAP', laboratorio: 'GENFAR PERU', presentacion: 'CAJA X 50 CAP', tipo: 'MEDICAMENTO', vencimiento: '2026-11-15', ean: '7702089140028' },
  { codigo: '309182', descripcion: 'EUCERIN OIL CONTROL TOQUE SECO FPS50+ 50ML', laboratorio: 'BEIERSDORF', presentacion: 'FRASCO 50ML', tipo: 'DERMOCOSMETICA', vencimiento: '2027-02-28', ean: '4005800123456' },
  { codigo: '401129', descripcion: 'LOSARTAN POTASICO 50MG PORTUGAL CAJA X 100 TAB', laboratorio: 'LAB PORTUGAL', presentacion: 'CAJA X 100 TAB', tipo: 'MEDICAMENTO', vencimiento: '2026-12-01', ean: '' },
  { codigo: '502938', descripcion: 'PARACETAMOL 500MG FARMDUSTRIA CAJA X 100 TAB', laboratorio: 'FARMDUSTRIA', presentacion: 'CAJA X 100 TAB', tipo: 'MEDICAMENTO', vencimiento: '2026-11-10', ean: '' },
  { codigo: '607811', descripcion: 'GERIAPLUS ADVANCE VAINILLA LATA 900G', laboratorio: 'HERSIL S.A.', presentacion: 'LATA 900G', tipo: 'NUTRICIONAL', vencimiento: '2027-01-20', ean: '7756078110099' },
  { codigo: '701245', descripcion: 'TENSIOMETRO DIGITAL DE BRAZO OMRON HEM-7120', laboratorio: 'OMRON HEALTHCARE', presentacion: 'CAJA X 1 UNID', tipo: 'EQUIPO MEDICO', vencimiento: '2029-12-31', ean: '' },
  { codigo: '804412', descripcion: 'IBUPROFENO 400MG MEDIFARMA CAJA X 100 TAB', laboratorio: 'MEDIFARMA', presentacion: 'CAJA X 100 TAB', tipo: 'MEDICAMENTO', vencimiento: '2026-11-25', ean: '' },
  { codigo: '809310', descripcion: 'APRONAX 550MG TABLETAS RECUBIERTAS CAJA X 120', laboratorio: 'BAYER', presentacion: 'CAJA X 120 TAB', tipo: 'MEDICAMENTO', vencimiento: '2027-01-12', ean: '' },
  { codigo: '901154', descripcion: 'BISMUTOL SUSPENSION ORAL FRASCO X 340ML', laboratorio: 'TEVA PERU', presentacion: 'FRASCO 340ML', tipo: 'MEDICAMENTO', vencimiento: '2026-12-18', ean: '' },
  { codigo: '905520', descripcion: 'AZITROMICINA 500MG TABLETAS CAJA X 3', laboratorio: 'IQFARMA', presentacion: 'CAJA X 3 TAB', tipo: 'MEDICAMENTO', vencimiento: '2026-11-22', ean: '' },
  { codigo: '908831', descripcion: 'TIRAS REACTIVAS ACCU-CHEK INSTANT CAJA X 50', laboratorio: 'ROCHE DIABETES', presentacion: 'CAJA X 50 TIRAS', tipo: 'EQUIPO MEDICO', vencimiento: '2026-11-08', ean: '' }
];

const MOTIVOS_CANJE = [
  'Próximo a vencer (Corto vencimiento)',
  'Producto vencido',
  'Deterioro de empaque / Blíster dañado',
  'Retiro de mercado / Alerta DIGEMID',
  'Sobrestock acordado con laboratorio',
  'Falla de fábrica / Equipo defectuoso'
];

const ESTADOS_CANJE = ['Pendiente', 'Enviado', 'En proceso', 'Aprobado', 'Rechazado', 'Canjeado'];

// ============================================================================
// 4. COMPONENTE PRINCIPAL EXPORTADO
// ============================================================================
export default function App() {
  return (
    <SafeErrorBoundary>
      <FarmaCanjesMain />
    </SafeErrorBoundary>
  );
}

function FarmaCanjesMain() {
  // --- NAVEGACIÓN Y TEMA ---
  const [activeTab, setActiveTab] = useState('nuevo');
  const [darkMode, setDarkMode] = useState(false);
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  // --- SONIDO Y FARMACIA ---
  const [soundEnabled, setSoundEnabled] = useState(true);
  const [soundVolume, setSoundVolume] = useState(0.65);
  const [pharmacyName, setPharmacyName] = useState('Farmacia Central - Sede Principal');

  // --- ESTADO DE LIBRERÍAS CDN ---
  const [libsLoaded, setLibsLoaded] = useState({ xlsx: false, jspdf: false });
  const [isProcessingDoc, setIsProcessingDoc] = useState(false);

  // --- NOTIFICACIONES TOAST ---
  const [toasts, setToasts] = useState([]);

  // --- DATOS PERSISTENTES ---
  const [catalog, setCatalog] = useState(() => {
    try {
      const saved = localStorage.getItem('fc_catalog_v3');
      return saved ? JSON.parse(saved) : INITIAL_CATALOG;
    } catch { return INITIAL_CATALOG; }
  });

  // Diccionario de Aprendizaje Permanente: EAN-13 (13 dígitos) -> Código Interno (6 dígitos)
  const [barcodeMap, setBarcodeMap] = useState(() => {
    try {
      const saved = localStorage.getItem('fc_barcode_map_v3');
      if (saved) return JSON.parse(saved);
      const initialMap = {};
      INITIAL_CATALOG.forEach(p => { if (p.ean) initialMap[p.ean] = p.codigo; });
      return initialMap;
    } catch { return {}; }
  });

  // Lista actual de productos del canje en curso
  const [cart, setCart] = useState(() => {
    try {
      const saved = localStorage.getItem('fc_current_cart_v3');
      return saved ? JSON.parse(saved) : [];
    } catch { return []; }
  });

  // Historial de solicitudes de canje
  const [history, setHistory] = useState(() => {
    try {
      const saved = localStorage.getItem('fc_history_v3');
      return saved ? JSON.parse(saved) : [
        {
          id: 'CANJE-001',
          fecha: '2026-10-30',
          estado: 'Enviado',
          totalEnteros: 3,
          totalFracciones: 20,
          items: [
            {
              uid: 1001,
              codigo: '104521',
              ean: '7750104521001',
              descripcion: 'PANADOL FORTE 500MG/65MG CAJA X 100 TAB',
              laboratorio: 'GSK CONSUMER',
              presentacion: 'CAJA X 100 TAB',
              enteros: 3,
              fracciones: 20,
              vencimiento: '2026-11-30',
              motivo: 'Próximo a vencer (Corto vencimiento)',
              observaciones: 'Blísters sellados en perfecto estado'
            }
          ]
        }
      ];
    } catch { return []; }
  });

  const [editingRequestId, setEditingRequestId] = useState(null);

  // --- ESTADOS DE ESCANEO Y APRENDIZAJE EAN-13 (OPCIÓN 2) ---
  const [barcodeInput, setBarcodeInput] = useState('');
  const [unrecognizedBarcode, setUnrecognizedBarcode] = useState(null);
  const [cameraModalOpen, setCameraModalOpen] = useState(false);
  const videoRef = useRef(null);
  const searchInputRef = useRef(null);
  const enterosInputRef = useRef(null);

  // --- ESTADOS DEL BUSCADOR Y FORMULARIO ---
  const [searchQuery, setSearchQuery] = useState('');
  const [showSuggestions, setShowSuggestions] = useState(false);
  const [selectedProduct, setSelectedProduct] = useState(null);
  const [justSelectedAnim, setJustSelectedAnim] = useState(false);

  // Campos de los 2 cuadritos independientes (Enteros y Fracciones) + Vencimiento
  const [formEnteros, setFormEnteros] = useState(1);
  const [formFracciones, setFormFracciones] = useState(0);
  const [formVencimiento, setFormVencimiento] = useState('');
  const [formMotivo, setFormMotivo] = useState(MOTIVOS_CANJE[0]);
  const [formObs, setFormObs] = useState('');

  // Modal de registro manual
  const [manualModalOpen, setManualModalOpen] = useState(false);
  const [manualForm, setManualForm] = useState({
    codigo: '',
    descripcion: '',
    laboratorio: '',
    presentacion: 'CAJA X 100 TAB',
    tipo: 'MEDICAMENTO',
    vencimiento: '',
    ean: ''
  });

  // Estado de subida de Excel
  const [uploadProgress, setUploadProgress] = useState(0);
  const [isUploading, setIsUploading] = useState(false);
  const [catalogFilter, setCatalogFilter] = useState('');

  // --- PERSISTENCIA AUTOMÁTICA EN LOCALSTORAGE ---
  useEffect(() => {
    try { localStorage.setItem('fc_catalog_v3', JSON.stringify(catalog)); } catch (e) {}
  }, [catalog]);

  useEffect(() => {
    try { localStorage.setItem('fc_barcode_map_v3', JSON.stringify(barcodeMap)); } catch (e) {}
  }, [barcodeMap]);

  useEffect(() => {
    try { localStorage.setItem('fc_current_cart_v3', JSON.stringify(cart)); } catch (e) {}
  }, [cart]);

  useEffect(() => {
    try { localStorage.setItem('fc_history_v3', JSON.stringify(history)); } catch (e) {}
  }, [history]);

  useEffect(() => {
    soundFX.enabled = soundEnabled;
    soundFX.volume = soundVolume;
  }, [soundEnabled, soundVolume]);

  // --- INYECCIÓN SEGURA DE LIBRERÍAS CDN (XLSX & JSPDF) ---
  useEffect(() => {
    const loadScript = (src, id) =>
      new Promise((resolve) => {
        if (document.getElementById(id)) {
          resolve(true);
          return;
        }
        const script = document.createElement('script');
        script.id = id;
        script.src = src;
        script.async = true;
        script.onload = () => resolve(true);
        script.onerror = () => resolve(false);
        document.body.appendChild(script);
      });

    async function initLibs() {
      const xlsxOk = await loadScript('https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js', 'xlsx-cdn');
      const jspdfOk = await loadScript('https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js', 'jspdf-cdn');
      if (jspdfOk) {
        await loadScript('https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js', 'autotable-cdn');
      }
      setLibsLoaded({
        xlsx: !!window.XLSX || xlsxOk,
        jspdf: !!(window.jspdf && window.jspdf.jsPDF) || jspdfOk
      });
    }
    initLibs();
  }, []);

  // --- NOTIFICACIONES VISUALES ---
  const notify = (msg, type = 'success') => {
    const id = Date.now() + Math.random();
    setToasts(prev => [...prev, { id, msg, type }]);
    setTimeout(() => {
      setToasts(prev => prev.filter(t => t.id !== id));
    }, 4200);
  };

  // ============================================================================
  // LÓGICA DE APRENDIZAJE DE CÓDIGO DE BARRAS EN 1 TOQUE (OPCIÓN 2 EAN-13)
  // ============================================================================
  const handleProcessBarcode = (rawCode) => {
    const code = String(rawCode || '').trim();
    if (!code) return;

    const mappedInternalCode = barcodeMap[code];
    let foundProduct = null;

    if (mappedInternalCode) {
      foundProduct = catalog.find(p => String(p.codigo).trim() === String(mappedInternalCode).trim());
    }
    if (!foundProduct) {
      foundProduct = catalog.find(p => String(p.ean || '').trim() === code || String(p.codigo).trim() === code);
    }

    if (foundProduct) {
      // ¡Reconocido en menos de 1 segundo!
      setUnrecognizedBarcode(null);
      setBarcodeInput('');
      handleSelectProduct(foundProduct);
      notify(`⚡ Código ${code} reconocido al instante: ${foundProduct.descripcion}`, 'success');
    } else {
      // PASO 1: Capturar el código no reconocido sin borrarlo y abrir el buscador por nombre
      soundFX.playError();
      setUnrecognizedBarcode(code);
      setBarcodeInput(code);
      setShowSuggestions(true);
      notify(`Código ${code} no vinculado. Busca el producto para asociarlo una sola vez.`, 'warning');
      setTimeout(() => {
        if (searchInputRef.current) {
          searchInputRef.current.focus();
        }
      }, 100);
    }
  };

  // PASO 2 y 3: Vincular el EAN-13 con el código interno de 6 dígitos y guardarlo en memoria
  const handleLinkBarcodeAndSelect = (product, e) => {
    if (e) e.stopPropagation();
    const eanToLink = unrecognizedBarcode;
    if (!eanToLink) {
      handleSelectProduct(product);
      return;
    }

    // Guardar en la memoria del teléfono (localStorage barcodeMap)
    setBarcodeMap(prev => ({
      ...prev,
      [eanToLink]: product.codigo
    }));

    // Guardar también en el catálogo local para exportar con la columna Codigo_Barra
    setCatalog(prev => prev.map(item =>
      String(item.codigo) === String(product.codigo)
        ? { ...item, ean: eanToLink }
        : item
    ));

    const updatedProduct = { ...product, ean: eanToLink };
    setUnrecognizedBarcode(null);
    setBarcodeInput('');
    handleSelectProduct(updatedProduct);
    soundFX.playExcelSuccess();
    notify(`🔗 ¡Asociado! EAN ${eanToLink} = Cód. ${product.codigo}. La próxima vez que escanees esta caja la reconocerá en <1 seg.`, 'success');
  };

  // Escaneo con cámara del celular
  useEffect(() => {
    let stream = null;
    let intervalId = null;

    if (cameraModalOpen) {
      navigator.mediaDevices?.getUserMedia({ video: { facingMode: 'environment' } })
        .then(s => {
          stream = s;
          if (videoRef.current) {
            videoRef.current.srcObject = s;
            videoRef.current.play().catch(() => {});
          }
          if ('BarcodeDetector' in window) {
            const detector = new window.BarcodeDetector({ formats: ['ean_13', 'ean_8', 'code_128', 'upc_a'] });
            intervalId = setInterval(async () => {
              if (videoRef.current && videoRef.current.readyState === 4) {
                try {
                  const barcodes = await detector.detect(videoRef.current);
                  if (barcodes.length > 0) {
                    const detectedCode = barcodes[0].rawValue;
                    setCameraModalOpen(false);
                    handleProcessBarcode(detectedCode);
                  }
                } catch {}
              }
            }, 350);
          }
        })
        .catch(() => {
          notify('No se pudo abrir la cámara. Puedes usar el lector o el botón Simular Caja Nueva.', 'error');
          setCameraModalOpen(false);
        });
    }

    return () => {
      if (intervalId) clearInterval(intervalId);
      if (stream) stream.getTracks().forEach(t => t.stop());
    };
  }, [cameraModalOpen]);

  // ============================================================================
  // BÚSQUEDA INTELIGENTE (ADMITE NOMBRES INCOMPLETOS Y ERRORES PEQUEÑOS)
  // ============================================================================
  const normalizeStr = (str) =>
    String(str || '')
      .toLowerCase()
      .normalize('NFD')
      .replace(/[\u0300-\u036f]/g, '')
      .trim();

  const filteredSuggestions = useMemo(() => {
    const q = normalizeStr(searchQuery);
    if (!q) return catalog.slice(0, 10);

    const tokens = q.split(/\s+/).filter(Boolean);

    return catalog
      .map(item => {
        const searchable = normalizeStr(`${item.codigo} ${item.descripcion} ${item.laboratorio} ${item.presentacion} ${item.ean || ''}`);
        let score = 0;

        if (normalizeStr(item.codigo).includes(q) || normalizeStr(item.ean).includes(q)) score += 100;
        if (normalizeStr(item.descripcion).startsWith(q)) score += 60;

        let matchedTokens = 0;
        tokens.forEach(tok => {
          if (searchable.includes(tok)) {
            matchedTokens++;
            score += 25;
          } else if (tok.length >= 4) {
            const sub = tok.slice(0, tok.length - 1);
            if (searchable.includes(sub)) {
              matchedTokens += 0.7;
              score += 10;
            }
          }
        });

        return { item, score, matchedTokens };
      })
      .filter(res => res.score > 0 && res.matchedTokens >= tokens.length * 0.6)
      .sort((a, b) => b.score - a.score)
      .slice(0, 15)
      .map(res => res.item);
  }, [catalog, searchQuery]);

  // Seleccionar medicamento y autocompletar
  const handleSelectProduct = (prod) => {
    soundFX.playSelect();
    setSelectedProduct(prod);
    setSearchQuery(prod.descripcion);
    setShowSuggestions(false);
    setFormEnteros(1);
    setFormFracciones(0);
    setFormVencimiento(prod.vencimiento || '2026-12-31');
    setFormMotivo(MOTIVOS_CANJE[0]);
    setFormObs('');
    setJustSelectedAnim(true);
    setTimeout(() => setJustSelectedAnim(false), 650);
    setTimeout(() => {
      if (enterosInputRef.current) {
        enterosInputRef.current.focus();
        enterosInputRef.current.select();
      }
    }, 90);
  };

  // ============================================================================
  // AGREGAR AL LISTADO DE CANJE (EVITA DUPLICADOS ACUMULANDO CANTIDADES)
  // ============================================================================
  const handleAddProductToCart = (e) => {
    if (e) e.preventDefault();
    if (!selectedProduct) {
      soundFX.playError();
      notify('Primero selecciona un medicamento o producto del buscador.', 'error');
      return;
    }

    const ent = Math.max(0, parseInt(formEnteros, 10) || 0);
    const frac = Math.max(0, parseInt(formFracciones, 10) || 0);

    if (ent === 0 && frac === 0) {
      soundFX.playError();
      notify('Ingresa al menos 1 unidad en ENTEROS o en FRACCIONES.', 'error');
      return;
    }

    if (!formVencimiento) {
      soundFX.playError();
      notify('Verifica o selecciona la Fecha de Vencimiento.', 'error');
      return;
    }

    const existingIndex = cart.findIndex(
      item => String(item.codigo) === String(selectedProduct.codigo) && item.vencimiento === formVencimiento
    );

    if (existingIndex >= 0) {
      const updated = [...cart];
      updated[existingIndex] = {
        ...updated[existingIndex],
        enteros: (parseInt(updated[existingIndex].enteros, 10) || 0) + ent,
        fracciones: (parseInt(updated[existingIndex].fracciones, 10) || 0) + frac,
        motivo: formMotivo,
        observaciones: formObs || updated[existingIndex].observaciones
      };
      setCart(updated);
      soundFX.playAdd();
      notify(`Cantidades acumuladas para ${selectedProduct.descripcion}`, 'success');
    } else {
      const newItem = {
        uid: Date.now() + Math.random(),
        codigo: selectedProduct.codigo,
        ean: selectedProduct.ean || '',
        descripcion: selectedProduct.descripcion,
        laboratorio: selectedProduct.laboratorio,
        presentacion: selectedProduct.presentacion || 'UNIDAD',
        enteros: ent,
        fracciones: frac,
        vencimiento: formVencimiento,
        motivo: formMotivo,
        observaciones: formObs
      };
      setCart(prev => [newItem, ...prev]);
      soundFX.playAdd();
      notify(`Agregado: ${selectedProduct.descripcion} (${ent} Ent / ${frac} Frac)`, 'success');
    }

    setSelectedProduct(null);
    setSearchQuery('');
    setFormEnteros(1);
    setFormFracciones(0);
    setFormObs('');
  };

  const handleUpdateCartQty = (uid, field, value) => {
    const num = Math.max(0, parseInt(value, 10) || 0);
    setCart(prev => prev.map(item => item.uid === uid ? { ...item, [field]: num } : item));
  };

  const handleRemoveFromCart = (uid) => {
    soundFX.playDelete();
    setCart(prev => prev.filter(item => item.uid !== uid));
    notify('Producto eliminado de la lista.', 'info');
  };

  // ============================================================================
  // CARGAR ARCHIVO EXCEL REAL DESDE EL ALMACENAMIENTO INTERNO
  // ============================================================================
  const handleExcelUpload = (e) => {
    const file = e.target.files?.[0];
    if (!file) return;

    if (!window.XLSX) {
      soundFX.playError();
      notify('El lector de Excel se está preparando, intenta en 2 segundos.', 'error');
      return;
    }

    setIsUploading(true);
    setUploadProgress(20);

    const reader = new FileReader();
    reader.onload = (evt) => {
      try {
        setUploadProgress(75);
        const data = new Uint8Array(evt.target.result);
        const workbook = window.XLSX.read(data, { type: 'array', cellDates: true });
        const firstSheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[firstSheetName];
        const jsonRows = window.XLSX.utils.sheet_to_json(worksheet, { defval: '' });

        if (!jsonRows || jsonRows.length === 0) {
          throw new Error('El archivo Excel está vacío.');
        }

        const newBarcodeMap = { ...barcodeMap };

        const parsedCatalog = jsonRows.map((row, idx) => {
          const keys = Object.keys(row);
          const findCol = (keywords) => {
            const foundKey = keys.find(k => keywords.some(kw => normalizeStr(k).includes(kw)));
            return foundKey ? row[foundKey] : '';
          };

          const rawCodigo = String(findCol(['codigo_interno', 'codigo', 'cod', 'id', 'item']) || (100000 + idx)).trim();
          const rawDesc = String(findCol(['descripcion', 'nombre', 'producto', 'medicamento', 'articulo', 'comercial']) || '').trim();
          const rawLab = String(findCol(['laboratorio', 'lab', 'marca', 'fabricante', 'proveedor']) || 'GENERAL').trim();
          const rawPres = String(findCol(['presentacion', 'unidad', 'empaque', 'forma']) || 'UNIDAD').trim();
          const rawTipo = String(findCol(['tipo', 'categoria', 'linea', 'familia']) || 'MEDICAMENTO').trim();
          const rawEan = String(findCol(['codigo_barra', 'barra', 'ean', 'gtin', 'upc']) || '').trim();

          let rawVenc = findCol(['vencimiento', 'vence', 'fecha_venc', 'expira', 'caducidad', 'fv']);
          let formattedVenc = '';
          if (rawVenc instanceof Date && !isNaN(rawVenc)) {
            formattedVenc = rawVenc.toISOString().split('T')[0];
          } else if (typeof rawVenc === 'string' && rawVenc.trim()) {
            const parts = rawVenc.trim().split(/[\/\-]/);
            if (parts.length === 3 && parts[2].length === 4) {
              formattedVenc = `${parts[2]}-${parts[1].padStart(2, '0')}-${parts[0].padStart(2, '0')}`;
            } else {
              formattedVenc = rawVenc.trim().slice(0, 10);
            }
          }

          // Conservar códigos EAN aprendidos previamente para este código interno
          const existingEanForCode = Object.keys(newBarcodeMap).find(k => String(newBarcodeMap[k]) === rawCodigo);
          const finalEan = rawEan || existingEanForCode || '';
          if (finalEan) newBarcodeMap[finalEan] = rawCodigo;

          return {
            codigo: rawCodigo,
            descripcion: rawDesc,
            laboratorio: rawLab,
            presentacion: rawPres,
            tipo: rawTipo,
            vencimiento: formattedVenc,
            ean: finalEan
          };
        }).filter(item => item.descripcion !== '');

        setCatalog(parsedCatalog);
        setBarcodeMap(newBarcodeMap);
        setUploadProgress(100);

        setTimeout(() => {
          setIsUploading(false);
          setUploadProgress(0);
          soundFX.playExcelSuccess();
          notify(`¡Excel cargado con éxito! ${parsedCatalog.length} productos disponibles.`, 'success');
        }, 350);
      } catch (err) {
        setIsUploading(false);
        setUploadProgress(0);
        soundFX.playError();
        notify('No se pudo leer el Excel. Asegúrate de subir un archivo .xlsx o .xls válido.', 'error');
      }
    };

    reader.readAsArrayBuffer(file);
    e.target.value = '';
  };

  // ============================================================================
  // DESCARGA FÍSICA DE DOCUMENTOS (PDF Y EXCEL) MEDIANTE BLOB
  // ============================================================================
  const triggerBlobDownload = (blob, filename) => {
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', filename);
    document.body.appendChild(link);
    link.click();
    setTimeout(() => {
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
    }, 400);
  };

  // Descargar el Excel Maestro con la nueva columna Codigo_Barra
  const handleDownloadUpdatedMasterExcel = () => {
    if (!catalog.length) return;
    try {
      const masterData = catalog.map(item => ({
        Codigo: item.codigo,
        Descripcion: item.descripcion,
        Laboratorio: item.laboratorio,
        Presentacion: item.presentacion,
        Tipo_Producto: item.tipo,
        Fecha_Vencimiento: item.vencimiento || '',
        Codigo_Barra: item.ean || ''
      }));

      const today = new Date().toISOString().slice(0, 10);
      const fileName = `MAESTRO_FARMACIA_CON_EAN_${today}.xlsx`;

      if (window.XLSX) {
        const ws = window.XLSX.utils.json_to_sheet(masterData);
        const wb = window.XLSX.utils.book_new();
        window.XLSX.utils.book_append_sheet(wb, ws, 'Maestro_Productos');
        const wbout = window.XLSX.write(wb, { bookType: 'xlsx', type: 'array' });
        const blob = new Blob([wbout], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
        triggerBlobDownload(blob, fileName);
        soundFX.playExcelSuccess();
        notify(`Maestro actualizado (${fileName}) descargado con la columna Codigo_Barra.`, 'success');
      } else {
        const headers = 'Codigo;Descripcion;Laboratorio;Presentacion;Tipo_Producto;Fecha_Vencimiento;Codigo_Barra';
        const rows = masterData.map(r => `${r.Codigo};"${r.Descripcion}";"${r.Laboratorio}";"${r.Presentacion}";${r.Tipo_Producto};${r.Fecha_Vencimiento};${r.Codigo_Barra}`).join('\n');
        const blob = new Blob(['\uFEFF' + headers + '\n' + rows], { type: 'text/csv;charset=utf-8;' });
        triggerBlobDownload(blob, `MAESTRO_FARMACIA_CON_EAN_${today}.csv`);
        soundFX.playExcelSuccess();
      }
    } catch (e) {
      soundFX.playError();
      notify('Error al exportar el Maestro.', 'error');
    }
  };

  const saveCurrentRequestToHistory = () => {
    if (cart.length === 0) {
      soundFX.playError();
      notify('Agrega productos al listado antes de guardar.', 'error');
      return null;
    }

    const totalEnteros = cart.reduce((acc, i) => acc + (parseInt(i.enteros, 10) || 0), 0);
    const totalFracciones = cart.reduce((acc, i) => acc + (parseInt(i.fracciones, 10) || 0), 0);
    const fechaHoy = new Date().toISOString().split('T')[0];

    if (editingRequestId) {
      const updatedReq = {
        id: editingRequestId,
        fecha: fechaHoy,
        estado: 'Pendiente',
        totalEnteros,
        totalFracciones,
        items: [...cart]
      };
      setHistory(prev => prev.map(h => h.id === editingRequestId ? updatedReq : h));
      setEditingRequestId(null);
      soundFX.playSave();
      notify(`Solicitud ${editingRequestId} actualizada.`, 'success');
      return updatedReq;
    } else {
      const nextNum = String(history.length + 1).padStart(3, '0');
      const newReq = {
        id: `CANJE-${nextNum}`,
        fecha: fechaHoy,
        estado: 'Pendiente',
        totalEnteros,
        totalFracciones,
        items: [...cart]
      };
      setHistory(prev => [newReq, ...prev]);
      soundFX.playSave();
      notify(`Solicitud ${newReq.id} guardada en el historial.`, 'success');
      return newReq;
    }
  };

  // Descargar Envío de Canje en Excel (.xlsx)
  const handleDownloadCanjeExcel = (customReq = null) => {
    const itemsToExport = customReq ? customReq.items : cart;
    if (!itemsToExport || itemsToExport.length === 0) {
      soundFX.playError();
      notify('La lista de canje está vacía.', 'error');
      return;
    }

    setIsProcessingDoc(true);
    const reqId = customReq ? customReq.id : (editingRequestId || `CANJE-${String(history.length + 1).padStart(3, '0')}`);
    const fecha = customReq ? customReq.fecha : new Date().toISOString().split('T')[0];

    setTimeout(() => {
      try {
        const rows = itemsToExport.map((item, idx) => {
          const ent = parseInt(item.enteros, 10) || 0;
          const frac = parseInt(item.fracciones, 10) || 0;
          let tipoUnidad = 'ENTERO';
          if (ent > 0 && frac > 0) tipoUnidad = 'ENTERO Y FRACCIÓN';
          else if (frac > 0) tipoUnidad = 'FRACCIÓN';

          return {
            'N° Solicitud': reqId,
            'Fecha Envío': fecha,
            'Ítem': idx + 1,
            'Código': item.codigo,
            'Código Barra (EAN)': item.ean || '',
            'Descripción': item.descripcion,
            'Laboratorio': item.laboratorio,
            'Enteros': ent,
            'Fracciones': frac,
            'Tipo Unidad': tipoUnidad,
            'Fecha Vencimiento': item.vencimiento || '-',
            'Motivo del Canje': item.motivo || '-',
            'Observaciones': item.observaciones || '-'
          };
        });

        const fileName = `${reqId}_${fecha}.xlsx`;

        if (window.XLSX) {
          const ws = window.XLSX.utils.json_to_sheet(rows);
          const wb = window.XLSX.utils.book_new();
          window.XLSX.utils.book_append_sheet(wb, ws, 'Envio_Canje');
          const wbout = window.XLSX.write(wb, { bookType: 'xlsx', type: 'array' });
          const blob = new Blob([wbout], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });
          triggerBlobDownload(blob, fileName);
        } else {
          const headers = Object.keys(rows[0]).join(';');
          const csvBody = rows.map(r => Object.values(r).map(v => `"${String(v).replace(/"/g, '""')}"`).join(';')).join('\n');
          const blob = new Blob(['\uFEFF' + headers + '\n' + csvBody], { type: 'text/csv;charset=utf-8;' });
          triggerBlobDownload(blob, `${reqId}_${fecha}.csv`);
        }

        setIsProcessingDoc(false);
        soundFX.playExcelSuccess();
        notify(`Archivo Excel (${fileName}) descargado en tu dispositivo.`, 'success');
      } catch (err) {
        setIsProcessingDoc(false);
        soundFX.playError();
        notify('Error al generar el Excel.', 'error');
      }
    }, 250);
  };

  // Generar y Descargar Envío de Canje en PDF
  const handleGenerateCanjePDF = (customReq = null, autoSaveToHistory = true) => {
    const itemsToExport = customReq ? customReq.items : cart;
    if (!itemsToExport || itemsToExport.length === 0) {
      soundFX.playError();
      notify('Agrega productos antes de generar el PDF.', 'error');
      return;
    }

    setIsProcessingDoc(true);
    let savedReq = customReq;
    if (!customReq && autoSaveToHistory) {
      savedReq = saveCurrentRequestToHistory();
    }

    const reqId = savedReq ? savedReq.id : `CANJE-${String(history.length + 1).padStart(3, '0')}`;
    const fecha = savedReq ? savedReq.fecha : new Date().toISOString().split('T')[0];
    const totalEnt = itemsToExport.reduce((a, b) => a + (parseInt(b.enteros, 10) || 0), 0);
    const totalFrac = itemsToExport.reduce((a, b) => a + (parseInt(b.fracciones, 10) || 0), 0);

    setTimeout(() => {
      try {
        if (window.jspdf && window.jspdf.jsPDF) {
          const { jsPDF } = window.jspdf;
          const doc = new jsPDF({ orientation: 'landscape', unit: 'mm', format: 'a4' });

          doc.setFillColor(5, 150, 105);
          doc.rect(0, 0, 297, 30, 'F');

          doc.setTextColor(255, 255, 255);
          doc.setFontSize(17);
          doc.setFont('helvetica', 'bold');
          doc.text('GUÍA OFICIAL DE ENVÍO DE CANJE FARMACÉUTICO', 14, 13);

          doc.setFontSize(10);
          doc.setFont('helvetica', 'normal');
          doc.text(`Establecimiento: ${pharmacyName}   |   N° Solicitud: ${reqId}   |   Fecha de Envío: ${fecha}`, 14, 22);

          doc.setTextColor(30, 41, 59);
          doc.setFontSize(10);
          doc.setFont('helvetica', 'bold');
          doc.text(`Resumen: ${itemsToExport.length} Productos   |   Total Unidades Enteras: ${totalEnt}   |   Total Unidades Fracción: ${totalFrac}`, 14, 38);

          const tableBody = itemsToExport.map((item, idx) => {
            const ent = parseInt(item.enteros, 10) || 0;
            const frac = parseInt(item.fracciones, 10) || 0;
            let tipoUnidad = 'ENTERO';
            if (ent > 0 && frac > 0) tipoUnidad = 'ENT + FRAC';
            else if (frac > 0) tipoUnidad = 'FRACCIÓN';

            return [
              idx + 1,
              item.codigo,
              item.descripcion,
              item.laboratorio,
              ent,
              frac,
              tipoUnidad,
              item.vencimiento || '-',
              item.motivo || '-',
              item.observaciones || '-'
            ];
          });

          if (doc.autoTable) {
            doc.autoTable({
              startY: 43,
              head: [['#', 'Código', 'Descripción del Producto', 'Laboratorio', 'Entero', 'Fracción', 'Tipo Unid.', 'Vencimiento', 'Motivo del Canje', 'Observaciones']],
              body: tableBody,
              theme: 'grid',
              headStyles: { fillColor: [5, 150, 105], textColor: 255, fontSize: 8.5, fontStyle: 'bold' },
              bodyStyles: { fontSize: 8, textColor: [30, 41, 59] },
              alternateRowStyles: { fillColor: [240, 253, 244] },
              columnStyles: {
                0: { cellWidth: 8, halign: 'center' },
                1: { cellWidth: 20, fontStyle: 'bold' },
                2: { cellWidth: 68 },
                3: { cellWidth: 34 },
                4: { cellWidth: 15, halign: 'center', fontStyle: 'bold' },
                5: { cellWidth: 16, halign: 'center', fontStyle: 'bold' },
                6: { cellWidth: 22, halign: 'center' },
                7: { cellWidth: 23, halign: 'center' },
                8: { cellWidth: 42 },
                9: { cellWidth: 31 }
              }
            });
          }

          const finalY = (doc.lastAutoTable && doc.lastAutoTable.finalY) ? doc.lastAutoTable.finalY + 20 : 160;
          doc.setDrawColor(148, 163, 184);
          doc.line(35, finalY, 105, finalY);
          doc.line(185, finalY, 255, finalY);
          doc.setFontSize(9);
          doc.text('Firma y Sello Responsable Farmacia', 42, finalY + 6);
          doc.text('Conformidad Recepción / Droguería / Lab.', 188, finalY + 6);

          const pdfBlob = doc.output('blob');
          const fileName = `${reqId}_${fecha}.pdf`;
          triggerBlobDownload(pdfBlob, fileName);

          if (!customReq) setCart([]);
          setIsProcessingDoc(false);
          soundFX.playPDF();
          notify(`Documento PDF (${fileName}) guardado en tu almacenamiento interno.`, 'success');
        } else {
          setIsProcessingDoc(false);
          window.print();
        }
      } catch (err) {
        setIsProcessingDoc(false);
        soundFX.playError();
        notify('Error al generar el PDF.', 'error');
      }
    }, 300);
  };

  // Registrar producto manual
  const handleSaveManualProduct = (e) => {
    e.preventDefault();
    if (!manualForm.descripcion.trim()) {
      soundFX.playError();
      notify('Ingresa el nombre del medicamento o producto.', 'error');
      return;
    }
    const newProd = {
      codigo: manualForm.codigo.trim() || String(Math.floor(100000 + Math.random() * 900000)),
      descripcion: manualForm.descripcion.trim().toUpperCase(),
      laboratorio: manualForm.laboratorio.trim().toUpperCase() || 'PARTICULAR',
      presentacion: manualForm.presentacion.trim().toUpperCase() || 'UNIDAD',
      tipo: manualForm.tipo,
      vencimiento: manualForm.vencimiento || '2026-12-31',
      ean: manualForm.ean.trim() || unrecognizedBarcode || ''
    };

    if (newProd.ean) {
      setBarcodeMap(prev => ({ ...prev, [newProd.ean]: newProd.codigo }));
    }

    setCatalog(prev => [newProd, ...prev]);
    setManualModalOpen(false);
    setUnrecognizedBarcode(null);
    setBarcodeInput('');
    handleSelectProduct(newProd);
    soundFX.playAdd();
    notify(`Producto ${newProd.descripcion} creado y seleccionado.`, 'success');
    setManualForm({ codigo: '', descripcion: '', laboratorio: '', presentacion: 'CAJA X 100 TAB', tipo: 'MEDICAMENTO', vencimiento: '', ean: '' });
  };

  const stats = useMemo(() => ({
    total: history.length,
    pendientes: history.filter(h => h.estado === 'Pendiente').length,
    enviados: history.filter(h => h.estado === 'Enviado' || h.estado === 'En proceso').length,
    aprobados: history.filter(h => h.estado === 'Aprobado' || h.estado === 'Canjeado').length,
    rechazados: history.filter(h => h.estado === 'Rechazado').length,
    codigosAprendidos: Object.keys(barcodeMap).length
  }), [history, barcodeMap]);

  const totalCartEnteros = cart.reduce((acc, item) => acc + (parseInt(item.enteros, 10) || 0), 0);
  const totalCartFracciones = cart.reduce((acc, item) => acc + (parseInt(item.fracciones, 10) || 0), 0);

  return (
    <div className={`${darkMode ? 'dark bg-slate-950 text-slate-100' : 'bg-slate-100 text-slate-800'} min-h-screen flex flex-col md:flex-row font-sans transition-colors duration-200`}>
      
      {/* NOTIFICACIONES TOAST */}
      <div className="fixed top-4 right-4 z-50 flex flex-col gap-2 max-w-sm w-full pointer-events-none px-3">
        {toasts.map(t => (
          <div
            key={t.id}
            className={`pointer-events-auto flex items-center gap-3 px-4 py-3.5 rounded-2xl shadow-2xl border text-xs md:text-sm font-bold transition-all ${
              t.type === 'error'
                ? 'bg-red-600 text-white border-red-400'
                : t.type === 'warning'
                ? 'bg-amber-500 text-slate-950 border-amber-300'
                : 'bg-emerald-600 text-white border-emerald-400'
            }`}
          >
            <span className="text-lg">{t.type === 'error' ? '⚠️' : t.type === 'warning' ? '🏷️' : '✅'}</span>
            <span className="flex-1 leading-snug">{t.msg}</span>
          </div>
        ))}
      </div>

      {/* OVERLAY DE GENERACIÓN DE DOCUMENTO */}
      {isProcessingDoc && (
        <div className="fixed inset-0 z-50 bg-slate-900/70 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="bg-white dark:bg-slate-800 p-6 rounded-3xl shadow-2xl flex flex-col items-center max-w-xs w-full text-center border border-emerald-500/40">
            <div className="w-12 h-12 border-4 border-emerald-500 border-t-transparent rounded-full animate-spin mb-3"></div>
            <h3 className="font-extrabold text-base text-slate-800 dark:text-white">Generando Archivo...</h3>
            <p className="text-xs text-slate-500 dark:text-slate-400 mt-1">Descargando directamente a tu almacenamiento interno</p>
          </div>
        </div>
      )}

      {/* MENÚ LATERAL (SIDEBAR) */}
      <aside className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border-b md:border-b-0 md:border-r md:w-64 flex-shrink-0 flex flex-col justify-between z-30`}>
        <div>
          <div className="p-4 md:p-5 flex items-center justify-between border-b border-slate-200 dark:border-slate-800">
            <div className="flex items-center gap-3">
              <div className="w-11 h-11 rounded-2xl bg-gradient-to-br from-emerald-500 to-teal-700 flex items-center justify-center text-white font-black text-xl shadow-lg shadow-emerald-600/30">
                ✚
              </div>
              <div>
                <h1 className="font-black text-base tracking-tight text-emerald-600 dark:text-emerald-400 leading-none">
                  FARMA<span className="text-slate-800 dark:text-white">CANJES</span>
                </h1>
                <span className="text-[11px] font-semibold text-slate-400">Gestión & Escáner EAN-13</span>
              </div>
            </div>

            <button
              onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
              className="md:hidden px-3 py-1.5 rounded-xl bg-slate-100 dark:bg-slate-800 text-xs font-bold"
            >
              {mobileMenuOpen ? '✕ Cerrar' : '☰ Menú'}
            </button>
          </div>

          <nav className={`${mobileMenuOpen ? 'block' : 'hidden'} md:block p-3 space-y-1`}>
            {[
              { id: 'inicio', label: 'Inicio', icon: '📊' },
              { id: 'nuevo', label: 'Nuevo envío de canje', icon: '📦', badge: cart.length },
              { id: 'excel', label: 'Cargar Excel', icon: '📥' },
              { id: 'historial', label: 'Historial de canjes', icon: '🗂️', badge: stats.pendientes },
              { id: 'productos', label: 'Productos registrados', icon: '💊', badge: catalog.length },
              { id: 'config', label: 'Configuración', icon: '⚙️' }
            ].map(item => (
              <button
                key={item.id}
                onClick={() => {
                  soundFX.playSelect();
                  setActiveTab(item.id);
                  setMobileMenuOpen(false);
                }}
                className={`w-full flex items-center justify-between px-3.5 py-3 rounded-2xl text-sm font-bold transition-all ${
                  activeTab === item.id
                    ? 'bg-emerald-600 text-white shadow-md shadow-emerald-600/25'
                    : 'text-slate-600 dark:text-slate-300 hover:bg-emerald-50 dark:hover:bg-slate-800/70'
                }`}
              >
                <div className="flex items-center gap-3">
                  <span>{item.icon}</span>
                  <span>{item.label}</span>
                </div>
                {item.badge > 0 && (
                  <span className={`text-xs px-2 py-0.5 rounded-full font-black ${
                    activeTab === item.id ? 'bg-white text-emerald-700' : 'bg-emerald-100 dark:bg-emerald-900/50 text-emerald-700 dark:text-emerald-300'
                  }`}>
                    {item.badge}
                  </span>
                )}
              </button>
            ))}
          </nav>
        </div>

        <div className="hidden md:flex flex-col gap-2 p-4 border-t border-slate-200 dark:border-slate-800 text-xs">
          <div className="flex items-center justify-between">
            <span className="text-slate-500 font-semibold">Sonidos</span>
            <button
              onClick={() => setSoundEnabled(!soundEnabled)}
              className={`px-2.5 py-1 rounded-lg font-bold ${soundEnabled ? 'bg-emerald-100 text-emerald-700 dark:bg-emerald-900/40 dark:text-emerald-300' : 'bg-slate-200 text-slate-600'}`}
            >
              {soundEnabled ? '🔊 Activo' : '🔇 Mudo'}
            </button>
          </div>
          <div className="flex items-center justify-between">
            <span className="text-slate-500 font-semibold">Tema Visual</span>
            <button
              onClick={() => setDarkMode(!darkMode)}
              className="px-2.5 py-1 rounded-lg font-bold bg-slate-200 dark:bg-slate-800"
            >
              {darkMode ? '🌙 Oscuro' : '☀️ Claro'}
            </button>
          </div>
        </div>
      </aside>

      {/* CONTENIDO PRINCIPAL */}
      <main className="flex-1 flex flex-col min-w-0 overflow-x-hidden">
        
        {/* CABECERA SUPERIOR */}
        <header className={`${darkMode ? 'bg-slate-900/90 border-slate-800' : 'bg-white border-slate-200'} sticky top-0 z-20 backdrop-blur-md border-b px-4 md:px-8 py-3.5 flex flex-wrap items-center justify-between gap-3`}>
          <div>
            <h2 className="text-base md:text-xl font-black tracking-tight flex items-center gap-2">
              <span>SISTEMA DE GESTIÓN DE CANJES</span>
              {editingRequestId && (
                <span className="text-xs bg-amber-500 text-slate-950 px-2.5 py-0.5 rounded-full font-black">
                  Editando {editingRequestId}
                </span>
              )}
            </h2>
            <p className="text-xs text-slate-500 dark:text-slate-400">
              {pharmacyName} • Catálogo: <strong>{catalog.length}</strong> ítems • Cajas EAN-13 vinculadas: <strong>{stats.codigosAprendidos}</strong>
            </p>
          </div>

          <div className="flex items-center gap-2">
            <button
              onClick={() => setSoundEnabled(!soundEnabled)}
              className="p-2 rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-800 text-sm"
            >
              {soundEnabled ? '🔊' : '🔇'}
            </button>
            <button
              onClick={() => setDarkMode(!darkMode)}
              className="p-2 rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-800 text-sm"
            >
              {darkMode ? '🌙' : '☀️'}
            </button>
          </div>
        </header>

        <div className="p-4 md:p-7 max-w-7xl w-full mx-auto space-y-6">

          {/* ====================================================================
              VISTA 1: INICIO (DASHBOARD)
             ==================================================================== */}
          {activeTab === 'inicio' && (
            <div className="space-y-6">
              <div className="grid grid-cols-2 lg:grid-cols-5 gap-4">
                {[
                  { title: 'Total Canjes', val: stats.total, color: 'from-emerald-500 to-teal-600', icon: '📋' },
                  { title: 'Pendientes', val: stats.pendientes, color: 'from-amber-500 to-orange-500', icon: '⏳' },
                  { title: 'Enviados', val: stats.enviados, color: 'from-sky-500 to-blue-600', icon: '🚚' },
                  { title: 'Aprobados', val: stats.aprobados, color: 'from-green-500 to-emerald-600', icon: '✅' },
                  { title: 'Rechazados', val: stats.rechazados, color: 'from-rose-500 to-red-600', icon: '❌' }
                ].map((c, i) => (
                  <div key={i} className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-2xl p-4 shadow-sm flex items-center justify-between`}>
                    <div>
                      <p className="text-[11px] font-extrabold text-slate-400 uppercase">{c.title}</p>
                      <p className="text-2xl md:text-3xl font-black mt-1">{c.val}</p>
                    </div>
                    <div className={`w-11 h-11 rounded-2xl bg-gradient-to-br ${c.color} text-white flex items-center justify-center text-lg shadow`}>
                      {c.icon}
                    </div>
                  </div>
                ))}
              </div>

              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <button
                  onClick={() => setActiveTab('nuevo')}
                  className="text-left bg-gradient-to-br from-emerald-600 to-teal-700 text-white p-6 rounded-3xl shadow-lg hover:shadow-xl transition"
                >
                  <span className="text-3xl">📦</span>
                  <h3 className="text-lg font-black mt-2">Nuevo Envío de Canje</h3>
                  <p className="text-xs text-emerald-100 mt-1">
                    Escanea cajas con aprendizaje EAN-13 en 1 toque o busca por nombre.
                  </p>
                </button>

                <button
                  onClick={() => setActiveTab('excel')}
                  className={`text-left ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border p-6 rounded-3xl shadow-sm hover:border-emerald-500 transition`}
                >
                  <span className="text-3xl">📥</span>
                  <h3 className="text-lg font-black mt-2">Cargar Base de Datos Excel</h3>
                  <p className="text-xs text-slate-500 mt-1">
                    Sube tu archivo .xlsx o .xls desde el almacenamiento interno de tu celular o PC.
                  </p>
                </button>

                <button
                  onClick={handleDownloadUpdatedMasterExcel}
                  className={`text-left ${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border p-6 rounded-3xl shadow-sm hover:border-sky-500 transition`}
                >
                  <span className="text-3xl">🏷️</span>
                  <h3 className="text-lg font-black mt-2">Descargar Maestro + EAN</h3>
                  <p className="text-xs text-slate-500 mt-1">
                    Descarga tu Excel actualizado con la columna Codigo_Barra ({stats.codigosAprendidos} códigos aprendidos).
                  </p>
                </button>
              </div>
            </div>
          )}

          {/* ====================================================================
              VISTA 2: NUEVO ENVÍO DE CANJE (ESCANEO EAN-13 + 2 CUADRITOS + PDF/EXCEL)
             ==================================================================== */}
          {activeTab === 'nuevo' && (
            <div className="space-y-6">

              {/* SECCIÓN DE ESCANEO EAN-13 Y BÚSQUEDA INTELIGENTE */}
              <div className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-3xl p-4 md:p-6 shadow-sm`}>
                <div className="flex flex-col md:flex-row md:items-center justify-between gap-3 mb-3">
                  <div>
                    <span className="text-[11px] font-black uppercase tracking-wider text-emerald-600 dark:text-emerald-400">
                      ⚡ Asociación de Código de Barras en 1 Toque + Buscador Inteligente
                    </span>
                    <h3 className="text-base md:text-lg font-black">1. Escanear Caja o Buscar Medicamento por Nombre</h3>
                  </div>

                  <div className="flex flex-wrap items-center gap-2">
                    <button
                      type="button"
                      onClick={() => setCameraModalOpen(true)}
                      className="px-3.5 py-2 bg-sky-600 hover:bg-sky-500 text-white text-xs font-black rounded-xl shadow transition"
                    >
                      📷 Cámara Celular
                    </button>
                    <button
                      type="button"
                      onClick={() => {
                        const randomNewEan = '775' + Math.floor(1000000000 + Math.random() * 8999999999);
                        handleProcessBarcode(randomNewEan);
                      }}
                      className="px-3.5 py-2 bg-amber-500/15 hover:bg-amber-500/25 text-amber-800 dark:text-amber-300 border border-amber-500/40 text-xs font-black rounded-xl transition"
                    >
                      🧪 Simular Caja Nueva (13 dígitos)
                    </button>
                    <button
                      type="button"
                      onClick={() => {
                        setManualForm(prev => ({ ...prev, ean: unrecognizedBarcode || '' }));
                        setManualModalOpen(true);
                      }}
                      className="px-3 py-2 bg-slate-200 dark:bg-slate-800 text-xs font-bold rounded-xl"
                    >
                      + Producto Manual
                    </button>
                  </div>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-12 gap-3 items-center">
                  {/* Campo Lector de Código de Barras */}
                  <div className="md:col-span-5">
                    <form
                      onSubmit={(e) => {
                        e.preventDefault();
                        handleProcessBarcode(barcodeInput);
                      }}
                      className="relative flex items-center"
                    >
                      <span className="absolute left-3.5 text-slate-400 font-mono text-sm">║▌║</span>
                      <input
                        type="text"
                        value={barcodeInput}
                        onChange={(e) => setBarcodeInput(e.target.value)}
                        placeholder="Escanea código de barras (13 dígitos)..."
                        className={`w-full pl-11 pr-24 py-3.5 rounded-2xl border text-sm font-bold focus:outline-none focus:ring-2 focus:ring-emerald-500 ${
                          darkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-300 text-slate-900'
                        }`}
                      />
                      <button
                        type="submit"
                        className="absolute right-2 px-3 py-2 bg-emerald-600 hover:bg-emerald-500 text-white text-xs font-black rounded-xl"
                      >
                        Leer EAN
                      </button>
                    </form>
                  </div>

                  {/* Buscador por Nombre / Descripción / Laboratorio */}
                  <div className="md:col-span-7 relative">
                    <div className="relative flex items-center">
                      <span className="absolute left-4 text-emerald-600">🔍</span>
                      <input
                        ref={searchInputRef}
                        type="text"
                        value={searchQuery}
                        onFocus={() => setShowSuggestions(true)}
                        onChange={(e) => {
                          setSearchQuery(e.target.value);
                          setShowSuggestions(true);
                        }}
                        placeholder="Escribe el nombre del medicamento (ej. Panadol, Losartan, Eucerin)..."
                        className={`w-full pl-11 pr-10 py-3.5 rounded-2xl border-2 text-sm md:text-base font-bold focus:outline-none transition ${
                          unrecognizedBarcode
                            ? 'border-amber-500 bg-amber-50/40 dark:bg-amber-950/20 ring-4 ring-amber-500/20'
                            : 'border-emerald-500/70 focus:border-emerald-600 bg-white dark:bg-slate-800'
                        }`}
                      />
                      {searchQuery && (
                        <button
                          type="button"
                          onClick={() => {
                            setSearchQuery('');
                            setSelectedProduct(null);
                            setShowSuggestions(true);
                          }}
                          className="absolute right-3.5 text-slate-400 hover:text-slate-600 font-bold"
                        >
                          ✕
                        </button>
                      )}
                    </div>

                    {/* Sugerencias Automáticas con botón "Vincular y Agregar" */}
                    {showSuggestions && (
                      <div className={`${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-white border-slate-200'} absolute left-0 right-0 top-full mt-2 border rounded-2xl shadow-2xl max-h-80 overflow-y-auto z-40 divide-y divide-slate-100 dark:divide-slate-700`}>
                        <div className="px-4 py-2 bg-slate-50 dark:bg-slate-900/70 flex items-center justify-between text-[11px] font-bold text-slate-500">
                          <span>
                            {unrecognizedBarcode
                              ? `PASO 2: Toca "Vincular y Agregar" en el medicamento para guardar el EAN ${unrecognizedBarcode}`
                              : `Sugerencias del catálogo (${filteredSuggestions.length})`}
                          </span>
                          <button type="button" onClick={() => setShowSuggestions(false)}>Cerrar ✕</button>
                        </div>

                        {filteredSuggestions.length === 0 ? (
                          <div className="p-5 text-center space-y-2">
                            <p className="text-sm text-slate-500">No se encontró "{searchQuery}" en el Excel.</p>
                            <button
                              type="button"
                              onClick={() => {
                                setManualForm(prev => ({
                                  ...prev,
                                  descripcion: searchQuery.toUpperCase(),
                                  ean: unrecognizedBarcode || ''
                                }));
                                setManualModalOpen(true);
                                setShowSuggestions(false);
                              }}
                              className="px-4 py-2 bg-emerald-600 text-white text-xs font-bold rounded-xl"
                            >
                              + Registrar "{searchQuery}" manualmente
                            </button>
                          </div>
                        ) : (
                          filteredSuggestions.map(prod => (
                            <div
                              key={prod.codigo}
                              onClick={() => {
                                if (unrecognizedBarcode) handleLinkBarcodeAndSelect(prod);
                                else handleSelectProduct(prod);
                              }}
                              className="px-4 py-3 hover:bg-emerald-50/80 dark:hover:bg-slate-700/60 cursor-pointer transition flex items-center justify-between gap-3"
                            >
                              <div className="min-w-0 flex-1">
                                <div className="flex items-center gap-2 flex-wrap">
                                  <span className="px-2 py-0.5 text-[11px] font-black rounded bg-emerald-100 dark:bg-emerald-900/60 text-emerald-800 dark:text-emerald-300">
                                    Cód: {prod.codigo}
                                  </span>
                                  {prod.ean && (
                                    <span className="px-2 py-0.5 text-[10px] font-mono font-bold rounded bg-sky-100 dark:bg-sky-900/50 text-sky-700 dark:text-sky-300">
                                      EAN: {prod.ean}
                                    </span>
                                  )}
                                  <span className="text-xs font-bold text-slate-500">{prod.laboratorio}</span>
                                </div>
                                <p className="font-extrabold text-sm text-slate-800 dark:text-white truncate mt-0.5">
                                  {prod.descripcion}
                                </p>
                              </div>

                              {unrecognizedBarcode ? (
                                <button
                                  type="button"
                                  onClick={(e) => handleLinkBarcodeAndSelect(prod, e)}
                                  className="px-3.5 py-2 bg-amber-500 hover:bg-amber-400 text-slate-950 font-black text-xs rounded-xl shadow-md flex-shrink-0"
                                >
                                  🔗 Vincular y Agregar
                                </button>
                              ) : (
                                <span className="text-xs font-bold text-emerald-600 dark:text-emerald-400 flex-shrink-0">
                                  Elegir →
                                </span>
                              )}
                            </div>
                          ))
                        )}
                      </div>
                    )}
                  </div>
                </div>

                {/* PASO 1: BANNER ÁMBAR DE CÓDIGO NO VINCULADO */}
                {unrecognizedBarcode && (
                  <div className="mt-4 p-4 rounded-2xl bg-amber-500/15 border-2 border-amber-500 text-amber-950 dark:text-amber-200 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3">
                    <div className="flex items-start gap-3">
                      <span className="text-2xl">🏷️</span>
                      <div>
                        <p className="font-black text-sm md:text-base">
                          Código <span className="underline font-mono">{unrecognizedBarcode}</span> no vinculado. Busca el producto para asociarlo una sola vez.
                        </p>
                        <p className="text-xs opacity-90 mt-0.5">
                          Escribe las primeras letras del medicamento en el buscador y presiona <strong>"🔗 Vincular y Agregar"</strong>. Quedará guardado en el teléfono y en tu Excel.
                        </p>
                      </div>
                    </div>
                    <button
                      type="button"
                      onClick={() => {
                        setUnrecognizedBarcode(null);
                        setBarcodeInput('');
                      }}
                      className="px-3 py-1.5 rounded-xl bg-amber-900/20 text-xs font-bold shrink-0"
                    >
                      Cancelar
                    </button>
                  </div>
                )}
              </div>

              {/* FORMULARIO CON LOS 2 CUADRITOS INDEPENDIENTES (ENTEROS Y FRACCIONES) + FECHA DE VENCIMIENTO */}
              <form
                onSubmit={handleAddProductToCart}
                className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-3xl p-5 md:p-6 shadow-sm transition-all ${
                  justSelectedAnim ? 'ring-4 ring-emerald-500/50' : ''
                }`}
              >
                <div className="flex items-center justify-between mb-4 border-b border-slate-100 dark:border-slate-800 pb-3">
                  <h3 className="font-black text-sm md:text-base uppercase tracking-wide text-emerald-700 dark:text-emerald-400">
                    2. Datos Automáticos, Enteros, Fracciones y Vencimiento
                  </h3>
                  {selectedProduct && (
                    <span className="text-xs font-black px-3 py-1 rounded-full bg-emerald-100 dark:bg-emerald-900/60 text-emerald-800 dark:text-emerald-300">
                      ✓ Autocompletado
                    </span>
                  )}
                </div>

                {/* Datos autocompletados sin tener que escribirlos de nuevo */}
                <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3 mb-5">
                  <div className="p-3 rounded-2xl bg-slate-50 dark:bg-slate-800/70 border border-slate-200 dark:border-slate-700">
                    <span className="text-[11px] font-bold uppercase text-slate-400 block">Código Interno</span>
                    <p className="text-base font-black text-emerald-600 dark:text-emerald-400 mt-0.5">
                      {selectedProduct ? selectedProduct.codigo : '---'}
                    </p>
                  </div>
                  <div className="sm:col-span-2 p-3 rounded-2xl bg-slate-50 dark:bg-slate-800/70 border border-slate-200 dark:border-slate-700">
                    <span className="text-[11px] font-bold uppercase text-slate-400 block">Descripción del Producto</span>
                    <p className="text-sm md:text-base font-black truncate mt-0.5">
                      {selectedProduct ? selectedProduct.descripcion : 'Selecciona un medicamento arriba'}
                    </p>
                  </div>
                  <div className="p-3 rounded-2xl bg-slate-50 dark:bg-slate-800/70 border border-slate-200 dark:border-slate-700">
                    <span className="text-[11px] font-bold uppercase text-slate-400 block">Laboratorio / Presentación</span>
                    <p className="text-xs font-bold truncate mt-1">
                      {selectedProduct ? `${selectedProduct.laboratorio} • ${selectedProduct.presentacion}` : '---'}
                    </p>
                  </div>
                </div>

                {/* LOS 2 CUADRITOS INDEPENDIENTES (ENTEROS Y FRACCIONES) + FECHA DE VENCIMIENTO */}
                <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-5">
                  
                  {/* Cuadrito 1: ENTEROS */}
                  <div className="p-4 rounded-3xl bg-sky-50 dark:bg-sky-950/30 border-2 border-sky-400/80 flex flex-col justify-between">
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xs font-black uppercase text-sky-900 dark:text-sky-300">
                        📦 CANTIDAD ENTEROS
                      </span>
                      <span className="text-[11px] font-bold px-2 py-0.5 rounded bg-sky-200/80 dark:bg-sky-900 text-sky-900 dark:text-sky-200">
                        Cajas / Frascos
                      </span>
                    </div>
                    <div className="flex items-center gap-2.5">
                      <button
                        type="button"
                        onClick={() => setFormEnteros(Math.max(0, (parseInt(formEnteros, 10) || 0) - 1))}
                        className="w-12 h-12 rounded-2xl bg-white dark:bg-slate-800 border border-sky-300 dark:border-sky-700 text-xl font-black text-sky-700 dark:text-sky-300"
                      >
                        -
                      </button>
                      <input
                        ref={enterosInputRef}
                        type="number"
                        min="0"
                        value={formEnteros}
                        onChange={(e) => setFormEnteros(e.target.value)}
                        className="w-full text-center text-3xl font-black py-2 rounded-2xl bg-white dark:bg-slate-900 border-2 border-sky-400 text-sky-900 dark:text-sky-200 outline-none"
                      />
                      <button
                        type="button"
                        onClick={() => setFormEnteros((parseInt(formEnteros, 10) || 0) + 1)}
                        className="w-12 h-12 rounded-2xl bg-sky-600 hover:bg-sky-500 text-xl font-black text-white"
                      >
                        +
                      </button>
                    </div>
                  </div>

                  {/* Cuadrito 2: FRACCIONES */}
                  <div className="p-4 rounded-3xl bg-amber-50 dark:bg-amber-950/30 border-2 border-amber-400/80 flex flex-col justify-between">
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xs font-black uppercase text-amber-900 dark:text-amber-300">
                        💊 CANTIDAD FRACCIONES
                      </span>
                      <span className="text-[11px] font-bold px-2 py-0.5 rounded bg-amber-200/80 dark:bg-amber-900 text-amber-950 dark:text-amber-200">
                        Blísters / Tabletas
                      </span>
                    </div>
                    <div className="flex items-center gap-2.5">
                      <button
                        type="button"
                        onClick={() => setFormFracciones(Math.max(0, (parseInt(formFracciones, 10) || 0) - 1))}
                        className="w-12 h-12 rounded-2xl bg-white dark:bg-slate-800 border border-amber-300 dark:border-amber-700 text-xl font-black text-amber-800 dark:text-amber-300"
                      >
                        -
                      </button>
                      <input
                        type="number"
                        min="0"
                        value={formFracciones}
                        onChange={(e) => setFormFracciones(e.target.value)}
                        className="w-full text-center text-3xl font-black py-2 rounded-2xl bg-white dark:bg-slate-900 border-2 border-amber-400 text-amber-900 dark:text-amber-200 outline-none"
                      />
                      <button
                        type="button"
                        onClick={() => setFormFracciones((parseInt(formFracciones, 10) || 0) + 1)}
                        className="w-12 h-12 rounded-2xl bg-amber-500 hover:bg-amber-400 text-xl font-black text-slate-950"
                      >
                        +
                      </button>
                    </div>
                  </div>

                  {/* Cuadrito 3: FECHA DE VENCIMIENTO */}
                  <div className="p-4 rounded-3xl bg-emerald-50 dark:bg-emerald-950/30 border-2 border-emerald-500/70 flex flex-col justify-between">
                    <div className="flex items-center justify-between mb-2">
                      <span className="text-xs font-black uppercase text-emerald-900 dark:text-emerald-300">
                        📅 FECHA DE VENCIMIENTO
                      </span>
                      <span className="text-[11px] font-bold px-2 py-0.5 rounded bg-emerald-200/80 dark:bg-emerald-900 text-emerald-900 dark:text-emerald-200">
                        Verificado
                      </span>
                    </div>
                    <input
                      type="date"
                      value={formVencimiento}
                      onChange={(e) => setFormVencimiento(e.target.value)}
                      className="w-full px-4 py-3 rounded-2xl bg-white dark:bg-slate-900 border-2 border-emerald-400 text-base font-black outline-none"
                    />
                  </div>
                </div>

                {/* Motivo, Observaciones y Botón Agregar */}
                <div className="grid grid-cols-1 md:grid-cols-12 gap-3 items-end">
                  <div className="md:col-span-4">
                    <label className="block text-xs font-bold uppercase text-slate-500 mb-1">Motivo del Canje</label>
                    <select
                      value={formMotivo}
                      onChange={(e) => setFormMotivo(e.target.value)}
                      className={`w-full px-3.5 py-3 rounded-2xl border text-sm font-bold ${
                        darkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-300'
                      }`}
                    >
                      {MOTIVOS_CANJE.map((m, i) => <option key={i} value={m}>{m}</option>)}
                    </select>
                  </div>
                  <div className="md:col-span-5">
                    <label className="block text-xs font-bold uppercase text-slate-500 mb-1">Observaciones (Lote, Estado)</label>
                    <input
                      type="text"
                      value={formObs}
                      onChange={(e) => setFormObs(e.target.value)}
                      placeholder="Ej. Lote L2049, caja sellada..."
                      className={`w-full px-3.5 py-3 rounded-2xl border text-sm font-bold ${
                        darkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-300'
                      }`}
                    />
                  </div>
                  <div className="md:col-span-3">
                    <button
                      type="submit"
                      className="w-full py-3.5 px-5 bg-emerald-600 hover:bg-emerald-500 text-white font-black text-sm rounded-2xl shadow-lg shadow-emerald-600/30 transition"
                    >
                      ➕ AGREGAR AL CANJE
                    </button>
                  </div>
                </div>
              </form>

              {/* LISTADO ACTUAL Y BOTÓN GENERAR ENVÍO DE CANJE (PDF Y EXCEL) */}
              <div className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-3xl p-5 md:p-6 shadow-sm`}>
                <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-3 mb-4">
                  <div>
                    <h3 className="text-base md:text-lg font-black">
                      📋 Productos en esta Solicitud ({cart.length})
                    </h3>
                    <p className="text-xs text-slate-500">
                      Total Unidades: <strong className="text-sky-600">{totalCartEnteros} Enteros</strong> | <strong className="text-amber-600">{totalCartFracciones} Fracciones</strong>
                    </p>
                  </div>

                  {cart.length > 0 && (
                    <div className="flex flex-wrap items-center gap-2">
                      <button
                        type="button"
                        onClick={() => saveCurrentRequestToHistory()}
                        className="px-3.5 py-2.5 rounded-xl border border-emerald-500/50 text-emerald-700 dark:text-emerald-300 text-xs font-bold"
                      >
                        💾 Guardar Pendiente
                      </button>
                      <button
                        type="button"
                        onClick={() => handleDownloadCanjeExcel(null)}
                        className="px-4 py-2.5 rounded-xl bg-teal-700 hover:bg-teal-600 text-white text-xs font-black shadow"
                      >
                        📊 Exportar a Excel (.xlsx)
                      </button>
                      <button
                        type="button"
                        onClick={() => handleGenerateCanjePDF(null, true)}
                        className="px-5 py-2.5 rounded-xl bg-gradient-to-r from-emerald-600 to-green-600 hover:from-emerald-500 hover:to-green-500 text-white text-xs md:text-sm font-black shadow-lg shadow-emerald-600/30"
                      >
                        📄 GENERAR ENVÍO DE CANJE (PDF)
                      </button>
                    </div>
                  )}
                </div>

                {cart.length === 0 ? (
                  <div className="py-10 text-center border-2 border-dashed border-slate-200 dark:border-slate-800 rounded-2xl text-slate-400">
                    <p className="text-2xl mb-1">🛒</p>
                    <p className="font-bold text-sm">Aún no has agregado medicamentos a este envío</p>
                  </div>
                ) : (
                  <div className="overflow-x-auto">
                    <table className="w-full text-left border-collapse text-xs md:text-sm">
                      <thead>
                        <tr className="border-b border-slate-200 dark:border-slate-800 text-[11px] font-black uppercase text-slate-400">
                          <th className="py-2.5 px-3">Código</th>
                          <th className="py-2.5 px-3">Medicamento / Laboratorio</th>
                          <th className="py-2.5 px-3 text-center">Enteros</th>
                          <th className="py-2.5 px-3 text-center">Fracciones</th>
                          <th className="py-2.5 px-3">Vencimiento</th>
                          <th className="py-2.5 px-3">Motivo</th>
                          <th className="py-2.5 px-3 text-right">Quitar</th>
                        </tr>
                      </thead>
                      <tbody className="divide-y divide-slate-200/60 dark:divide-slate-800">
                        {cart.map(item => (
                          <tr key={item.uid}>
                            <td className="py-3 px-3 font-mono font-bold text-emerald-600 dark:text-emerald-400">
                              {item.codigo}
                            </td>
                            <td className="py-3 px-3">
                              <p className="font-extrabold">{item.descripcion}</p>
                              <p className="text-xs text-slate-400">{item.laboratorio}</p>
                            </td>
                            <td className="py-3 px-3 text-center">
                              <input
                                type="number"
                                min="0"
                                value={item.enteros}
                                onChange={(e) => handleUpdateCartQty(item.uid, 'enteros', e.target.value)}
                                className="w-16 text-center font-black py-1 rounded-xl border border-sky-400 bg-sky-50 dark:bg-sky-950/50"
                              />
                            </td>
                            <td className="py-3 px-3 text-center">
                              <input
                                type="number"
                                min="0"
                                value={item.fracciones}
                                onChange={(e) => handleUpdateCartQty(item.uid, 'fracciones', e.target.value)}
                                className="w-16 text-center font-black py-1 rounded-xl border border-amber-400 bg-amber-50 dark:bg-amber-950/50"
                              />
                            </td>
                            <td className="py-3 px-3 font-bold">{item.vencimiento}</td>
                            <td className="py-3 px-3 text-xs">{item.motivo}</td>
                            <td className="py-3 px-3 text-right">
                              <button
                                type="button"
                                onClick={() => handleRemoveFromCart(item.uid)}
                                className="p-1.5 rounded-lg bg-rose-500/10 text-rose-600 font-bold"
                              >
                                🗑️
                              </button>
                            </td>
                          </tr>
                        ))}
                      </tbody>
                    </table>
                  </div>
                )}
              </div>
            </div>
          )}

          {/* ====================================================================
              VISTA 3: CARGAR BASE DE DATOS EXCEL
             ==================================================================== */}
          {activeTab === 'excel' && (
            <div className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-3xl p-6 md:p-8 shadow-sm space-y-6`}>
              <div>
                <h3 className="text-xl font-black">📥 Cargar Base de Datos Excel desde tu Almacenamiento</h3>
                <p className="text-xs md:text-sm text-slate-500 mt-1">
                  Sube tu archivo Excel (.xlsx o .xls). Se conservarán automáticamente los códigos de barras EAN-13 que ya hayas aprendido en el teléfono.
                </p>
              </div>

              <label className="cursor-pointer block border-2 border-dashed border-emerald-500/60 hover:border-emerald-500 rounded-3xl p-10 text-center bg-emerald-50/40 dark:bg-emerald-950/10 transition">
                <input type="file" accept=".xlsx,.xls,.csv" onChange={handleExcelUpload} className="hidden" />
                <div className="text-4xl mb-3">📂</div>
                <p className="font-black text-base md:text-lg text-emerald-700 dark:text-emerald-400">
                  SELECCIONAR ARCHIVO EXCEL (.XLSX / .XLS)
                </p>
                <p className="text-xs text-slate-400 mt-1">Lectura instantánea de Código, Descripción, Laboratorio, Presentación y Vencimiento</p>
              </label>

              {isUploading && (
                <div className="w-full bg-slate-200 dark:bg-slate-800 h-3 rounded-full overflow-hidden">
                  <div className="bg-emerald-500 h-full transition-all" style={{ width: `${uploadProgress}%` }}></div>
                </div>
              )}

              <div className="pt-4 border-t border-slate-200 dark:border-slate-800 flex flex-wrap items-center justify-between gap-4">
                <div>
                  <h4 className="font-bold text-sm">Descargar Excel Maestro con Columna Codigo_Barra</h4>
                  <p className="text-xs text-slate-500">Guarda una copia de tu catálogo con todos los EAN-13 vinculados.</p>
                </div>
                <button
                  type="button"
                  onClick={handleDownloadUpdatedMasterExcel}
                  className="px-5 py-3 bg-sky-600 hover:bg-sky-500 text-white font-black text-xs rounded-2xl shadow"
                >
                  ⬇️ Descargar Maestro Actualizado (.xlsx)
                </button>
              </div>
            </div>
          )}

          {/* ====================================================================
              VISTA 4: HISTORIAL DE CANJES
             ==================================================================== */}
          {activeTab === 'historial' && (
            <div className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-3xl p-5 md:p-6 shadow-sm space-y-4`}>
              <h3 className="text-lg font-black">🗂️ Historial de Solicitudes de Canje ({history.length})</h3>
              <div className="space-y-3">
                {history.map(req => (
                  <div key={req.id} className="p-4 rounded-2xl border border-slate-200 dark:border-slate-800 flex flex-col md:flex-row md:items-center justify-between gap-3">
                    <div>
                      <div className="flex items-center gap-2.5 flex-wrap">
                        <span className="font-black text-base text-emerald-600">{req.id}</span>
                        <span className="text-xs text-slate-400">📅 {req.fecha}</span>
                        <select
                          value={req.estado}
                          onChange={(e) => {
                            const nuevo = e.target.value;
                            setHistory(prev => prev.map(h => h.id === req.id ? { ...h, estado: nuevo } : h));
                            notify(`Estado de ${req.id} actualizado a ${nuevo}`, 'success');
                          }}
                          className="text-xs font-bold px-2.5 py-1 rounded-full border border-emerald-500/40 bg-transparent"
                        >
                          {ESTADOS_CANJE.map(st => <option key={st} value={st} className="text-slate-900">{st}</option>)}
                        </select>
                      </div>
                      <p className="text-xs text-slate-500 mt-1">
                        {req.items.length} ítems • Enteros: <strong>{req.totalEnteros}</strong> • Fracciones: <strong>{req.totalFracciones}</strong>
                      </p>
                    </div>
                    <div className="flex flex-wrap items-center gap-2">
                      <button
                        onClick={() => {
                          setCart(req.items);
                          setEditingRequestId(req.id);
                          setActiveTab('nuevo');
                        }}
                        className="px-3 py-2 rounded-xl bg-amber-500/15 text-amber-700 dark:text-amber-300 text-xs font-bold"
                      >
                        ✏️ Modificar
                      </button>
                      <button
                        onClick={() => handleDownloadCanjeExcel(req)}
                        className="px-3 py-2 rounded-xl bg-teal-600 text-white text-xs font-bold"
                      >
                        📊 Excel
                      </button>
                      <button
                        onClick={() => handleGenerateCanjePDF(req, false)}
                        className="px-3.5 py-2 rounded-xl bg-emerald-600 text-white text-xs font-bold"
                      >
                        📄 PDF
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* ====================================================================
              VISTA 5: PRODUCTOS REGISTRADOS
             ==================================================================== */}
          {activeTab === 'productos' && (
            <div className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-3xl p-5 md:p-6 shadow-sm space-y-4`}>
              <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                <h3 className="text-lg font-black">💊 Productos Registrados ({catalog.length})</h3>
                <div className="flex items-center gap-2">
                  <input
                    type="text"
                    value={catalogFilter}
                    onChange={(e) => setCatalogFilter(e.target.value)}
                    placeholder="Buscar en catálogo..."
                    className={`px-3.5 py-2 rounded-xl border text-xs font-bold ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-50 border-slate-300'}`}
                  />
                  <button
                    onClick={handleDownloadUpdatedMasterExcel}
                    className="px-3.5 py-2 bg-sky-600 text-white text-xs font-bold rounded-xl"
                  >
                    ⬇️ Maestro + EAN
                  </button>
                </div>
              </div>

              <div className="overflow-x-auto max-h-96">
                <table className="w-full text-left border-collapse text-xs">
                  <thead className="sticky top-0 bg-slate-100 dark:bg-slate-800 font-black uppercase text-slate-500">
                    <tr>
                      <th className="py-2.5 px-3">Código</th>
                      <th className="py-2.5 px-3">Código Barra (EAN)</th>
                      <th className="py-2.5 px-3">Descripción</th>
                      <th className="py-2.5 px-3">Laboratorio</th>
                      <th className="py-2.5 px-3 text-right">Acción</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-200/60 dark:divide-slate-800">
                    {catalog
                      .filter(p => !catalogFilter || normalizeStr(`${p.codigo} ${p.descripcion} ${p.laboratorio} ${p.ean}`).includes(normalizeStr(catalogFilter)))
                      .slice(0, 80)
                      .map(prod => (
                        <tr key={prod.codigo}>
                          <td className="py-2.5 px-3 font-mono font-bold text-emerald-600">{prod.codigo}</td>
                          <td className="py-2.5 px-3 font-mono">{prod.ean || <span className="text-slate-400">Sin EAN</span>}</td>
                          <td className="py-2.5 px-3 font-bold">{prod.descripcion}</td>
                          <td className="py-2.5 px-3 text-slate-500">{prod.laboratorio}</td>
                          <td className="py-2.5 px-3 text-right">
                            <button
                              onClick={() => {
                                handleSelectProduct(prod);
                                setActiveTab('nuevo');
                              }}
                              className="px-2.5 py-1 bg-emerald-600 text-white font-bold rounded-lg"
                            >
                              Seleccionar
                            </button>
                          </td>
                        </tr>
                      ))}
                  </tbody>
                </table>
              </div>
            </div>
          )}

          {/* ====================================================================
              VISTA 6: CONFIGURACIÓN
             ==================================================================== */}
          {activeTab === 'config' && (
            <div className={`${darkMode ? 'bg-slate-900 border-slate-800' : 'bg-white border-slate-200'} border rounded-3xl p-6 shadow-sm space-y-4 max-w-xl`}>
              <h3 className="text-lg font-black">⚙️ Configuración</h3>
              <div>
                <label className="block text-xs font-bold uppercase text-slate-500 mb-1">Nombre de la Farmacia / Sucursal</label>
                <input
                  type="text"
                  value={pharmacyName}
                  onChange={(e) => setPharmacyName(e.target.value)}
                  className={`w-full px-3.5 py-2.5 rounded-xl border text-sm font-bold ${darkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-50 border-slate-300'}`}
                />
              </div>
              <div className="flex items-center justify-between pt-2">
                <span className="text-sm font-bold">Efectos de Sonido</span>
                <input
                  type="checkbox"
                  checked={soundEnabled}
                  onChange={(e) => setSoundEnabled(e.target.checked)}
                  className="w-5 h-5 accent-emerald-600"
                />
              </div>
              <div>
                <label className="block text-xs font-bold mb-1">Volumen ({Math.round(soundVolume * 100)}%)</label>
                <input
                  type="range"
                  min="0.1"
                  max="1"
                  step="0.1"
                  value={soundVolume}
                  onChange={(e) => setSoundVolume(parseFloat(e.target.value))}
                  className="w-full accent-emerald-600"
                />
              </div>
            </div>
          )}

        </div>
      </main>

      {/* MODAL DE CÁMARA ESCÁNER */}
      {cameraModalOpen && (
        <div className="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="bg-white dark:bg-slate-900 border border-slate-700 rounded-3xl max-w-md w-full p-5 space-y-4">
            <div className="flex items-center justify-between">
              <h3 className="font-black text-base">📷 Escanear Código EAN-13</h3>
              <button onClick={() => setCameraModalOpen(false)} className="px-3 py-1 rounded-xl bg-slate-200 dark:bg-slate-800 text-xs font-bold">
                Cerrar ✕
              </button>
            </div>
            <div className="relative rounded-2xl overflow-hidden bg-black aspect-video flex items-center justify-center">
              <video ref={videoRef} className="w-full h-full object-cover" playsInline muted />
              <div className="absolute inset-x-8 h-0.5 bg-red-500 animate-pulse"></div>
            </div>
            <p className="text-xs text-center text-slate-500">Apunta la cámara al código de barras de la caja.</p>
          </div>
        </div>
      )}

      {/* MODAL DE PRODUCTO MANUAL */}
      {manualModalOpen && (
        <div className="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-sm flex items-center justify-center p-4">
          <form onSubmit={handleSaveManualProduct} className="bg-white dark:bg-slate-900 border border-slate-700 rounded-3xl max-w-md w-full p-6 space-y-3">
            <h3 className="font-black text-lg">➕ Registrar Producto Manual</h3>
            <input
              type="text"
              placeholder="Código Interno (Ej. 901245)"
              value={manualForm.codigo}
              onChange={(e) => setManualForm({ ...manualForm, codigo: e.target.value })}
              className="w-full px-3.5 py-2.5 rounded-xl border text-sm bg-transparent"
            />
            <input
              type="text"
              required
              placeholder="Descripción del Medicamento *"
              value={manualForm.descripcion}
              onChange={(e) => setManualForm({ ...manualForm, descripcion: e.target.value })}
              className="w-full px-3.5 py-2.5 rounded-xl border text-sm bg-transparent"
            />
            <input
              type="text"
              placeholder="Laboratorio"
              value={manualForm.laboratorio}
              onChange={(e) => setManualForm({ ...manualForm, laboratorio: e.target.value })}
              className="w-full px-3.5 py-2.5 rounded-xl border text-sm bg-transparent"
            />
            <input
              type="text"
              placeholder="Código de Barras EAN-13 (Opcional)"
              value={manualForm.ean}
              onChange={(e) => setManualForm({ ...manualForm, ean: e.target.value })}
              className="w-full px-3.5 py-2.5 rounded-xl border text-sm bg-transparent"
            />
            <div className="flex justify-end gap-2 pt-2">
              <button type="button" onClick={() => setManualModalOpen(false)} className="px-4 py-2 rounded-xl bg-slate-200 dark:bg-slate-800 text-xs font-bold">
                Cancelar
              </button>
              <button type="submit" className="px-4 py-2 rounded-xl bg-emerald-600 text-white text-xs font-black">
                Guardar y Seleccionar
              </button>
            </div>
          </form>
        </div>
      )}

    </div>
  );
}
