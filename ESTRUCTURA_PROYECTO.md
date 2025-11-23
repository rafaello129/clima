# 📋 Estructura del Proyecto - App Clima

> **Documentación detallada para integración de Backend**
> 
> Este documento describe la arquitectura completa del frontend de la aplicación de clima, diseñada para ayudar a otro equipo o IA a desarrollar un backend compatible.

## 🏗️ Arquitectura General

### Stack Tecnológico
- **Framework**: React 19.1.1 con TypeScript 4.9.5
- **Build Tool**: Create React App (react-scripts 5.0.1)
- **UI Framework**: Material-UI (@mui/material 7.3.2)
- **Routing**: React Router DOM 7.9.3
- **Mapas**: Leaflet 1.9.4 + React-Leaflet 5.0.0
- **Gráficos**: Chart.js 4.5.0 + React-ChartJS-2 5.3.0
- **HTTP Client**: Axios 1.12.2
- **Animaciones**: Framer Motion 12.23.22
- **Gestión de Formularios**: React Hook Form 7.63.0
- **Fechas**: Day.js 1.11.18

### Configuración de Despliegue
- **Homepage**: `https://rafaello129.github.io/clima`
- **Basename del Router**: `/clima`
- **Despliegue**: GitHub Pages con GitHub Actions automático

---

## 📁 Estructura de Directorios

```
src/
├── api/                    # Capa de integración con API externa
│   ├── openWeather.ts     # Cliente API de OpenWeather
│   └── types.ts           # Tipos TypeScript para datos del clima
│
├── components/            # Componentes React reutilizables
│   ├── common/           # Componentes compartidos básicos
│   │   ├── CityForecastPreview.tsx
│   │   ├── ErrorMessage.tsx
│   │   └── LoadingSpinner.tsx
│   ├── layout/           # Componentes de layout
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── CityListMenu.tsx
│   │   └── ThemeToggleButton.tsx
│   ├── Forecast/         # Pronóstico del tiempo
│   │   ├── Forecast.tsx
│   │   └── Forecast.module.css
│   ├── SearchBar/        # Barra de búsqueda de ciudades
│   │   ├── SearchBar.tsx
│   │   └── SearchBar.module.css
│   ├── WeatherCard/      # Tarjeta principal de clima
│   │   ├── WeatherCard.tsx
│   │   └── WeatherCard.module.css
│   └── WeatherMap/       # Mapa interactivo
│       ├── WeatherMap.tsx
│       └── WeatherMap.module.css
│
├── contexts/             # React Context API
│   └── ThemeContext.tsx # Gestión de tema claro/oscuro
│
├── hooks/                # Custom React Hooks
│   ├── useLocalStorage.ts    # Persistencia en localStorage
│   └── useWeatherData.ts     # Gestión de datos del clima
│
├── pages/                # Páginas principales (Rutas)
│   ├── DashboardPage.tsx    # Página principal (/)
│   ├── MapPage.tsx          # Vista de mapa (/map)
│   └── SettingsPage.tsx     # Configuración (/settings)
│
├── services/             # Servicios y configuraciones
│   ├── theme.ts         # Temas de Material-UI
│   └── global.css       # Estilos globales
│
├── utils/                # Utilidades y helpers
│   ├── formatters.ts    # Funciones de formateo
│   └── recommendations.ts # Recomendaciones según el clima
│
├── App.tsx              # Componente raíz de la aplicación
├── index.tsx            # Punto de entrada
└── index.css            # Estilos base

public/
├── index.html           # HTML base
├── manifest.json        # PWA manifest
├── favicon.ico
├── logo192.png
└── logo512.png

Configuración:
├── package.json         # Dependencias y scripts
├── tsconfig.json        # Configuración TypeScript
└── .gitignore          # Archivos ignorados
```

---

## 🔌 Integración con API Externa (OpenWeather)

### Archivo: `src/api/openWeather.ts`

**API KEY Actual**: `826b37548275c1f2cda3cca800b7fd08`
**Base URL**: `https://api.openweathermap.org`

#### Endpoints Utilizados:

1. **Geocodificación** - Obtener coordenadas de una ciudad
   ```
   GET /geo/1.0/direct
   Params: { q: city, limit: 1, appid: API_KEY }
   ```

2. **Clima Actual** - Datos meteorológicos actuales
   ```
   GET /data/2.5/weather
   Params: { lat, lon, appid: API_KEY, units: 'metric', lang: 'es' }
   ```

3. **Pronóstico 5 Días** - Pronóstico extendido cada 3 horas
   ```
   GET /data/2.5/forecast
   Params: { lat, lon, appid: API_KEY, units: 'metric', lang: 'es' }
   ```

4. **Calidad del Aire** - Índice de calidad del aire (AQI)
   ```
   GET /data/2.5/air_pollution
   Params: { lat, lon, appid: API_KEY }
   ```

#### Función Principal:
```typescript
export const getWeatherAndForecast = async (city: string) => {
  // 1. Obtiene coordenadas de la ciudad (agregando ",MX" al final)
  const location = await getCoordsByCity(`${city},MX`);
  
  // 2. Solicita en paralelo: clima actual, pronóstico y calidad del aire
  const [currentWeather, forecast, airPollution] = await Promise.all([
    getCurrentWeather(lat, lon),
    get5DayForecast(lat, lon),
    getAirPollution(lat, lon),
  ]);
  
  // 3. Retorna objeto combinado
  return { location, currentWeather, forecast, airPollution };
}
```

---

## 📊 Tipos de Datos (TypeScript Interfaces)

### Archivo: `src/api/types.ts`

#### GeoLocation
```typescript
interface GeoLocation {
  name: string;        // Nombre de la ciudad
  lat: number;         // Latitud
  lon: number;         // Longitud
  country: string;     // Código de país (ej: "MX")
  state?: string;      // Estado (opcional)
}
```

#### CurrentWeatherData
```typescript
interface CurrentWeatherData {
  coord: { lon: number; lat: number };
  weather: Array<{
    id: number;           // ID de condición climática
    main: string;         // Grupo principal (Rain, Snow, Clear)
    description: string;  // Descripción detallada
    icon: string;         // Código de icono
  }>;
  main: {
    temp: number;         // Temperatura actual (°C)
    feels_like: number;   // Sensación térmica (°C)
    temp_min: number;     // Temperatura mínima (°C)
    temp_max: number;     // Temperatura máxima (°C)
    pressure: number;     // Presión atmosférica (hPa)
    humidity: number;     // Humedad relativa (%)
  };
  visibility: number;     // Visibilidad (metros)
  wind: {
    speed: number;        // Velocidad del viento (m/s)
    deg: number;          // Dirección del viento (grados)
  };
  clouds: {
    all: number;          // Nubosidad (%)
  };
  dt: number;             // Timestamp Unix
  sys: {
    country: string;
    sunrise: number;      // Timestamp Unix del amanecer
    sunset: number;       // Timestamp Unix del atardecer
  };
  timezone: number;       // Desplazamiento UTC (segundos)
  name: string;          // Nombre de la ciudad
}
```

#### ForecastData
```typescript
interface ForecastListItem {
  dt: number;            // Timestamp Unix
  main: {
    temp: number;        // Temperatura (°C)
    feels_like: number;  // Sensación térmica (°C)
    temp_min: number;
    temp_max: number;
    humidity: number;
  };
  weather: Array<{...}>;
  clouds: { all: number };
  wind: { speed: number };
  pop: number;           // Probabilidad de precipitación (0-1)
  dt_txt: string;        // Fecha/hora en formato texto
}

interface ForecastData {
  list: ForecastListItem[];  // Array de pronósticos cada 3 horas
  city: {
    name: string;
    country: string;
    sunrise: number;
    sunset: number;
    timezone: number;
  };
}
```

#### AirPollutionData
```typescript
interface AirPollutionData {
  list: Array<{
    main: { 
      aqi: 1 | 2 | 3 | 4 | 5;  // Índice de calidad del aire
      // 1=Bueno, 2=Aceptable, 3=Moderado, 4=Malo, 5=Muy Malo
    };
  }>;
}
```

#### CombinedWeatherData (Usado internamente)
```typescript
interface CombinedWeatherData {
  location: GeoLocation;
  currentWeather: CurrentWeatherData;
  forecast: ForecastData;
  airPollution: AirPollutionData;
  timestamp: number;  // Timestamp de cuando se obtuvieron los datos
}
```

---

## 🔧 Hooks Personalizados

### 1. useWeatherData Hook
**Archivo**: `src/hooks/useWeatherData.ts`

**Propósito**: Gestión centralizada del estado de datos meteorológicos de múltiples ciudades.

**Estado Interno**:
```typescript
{
  data: Map<string, CombinedWeatherData>,  // Datos de cada ciudad
  loading: Set<string>,                     // Ciudades en carga
  error: Map<string, string>,               // Errores por ciudad
  cities: string[]                          // Lista de ciudades guardadas
}
```

**Funciones Exportadas**:
- `addCity(city: string)` - Añade una ciudad y obtiene sus datos
- `removeCity(city: string)` - Elimina una ciudad
- `refreshCity(city: string)` - Actualiza datos de una ciudad
- `clearAllCities()` - Elimina todas las ciudades

**Características**:
- Cache de 10 minutos (`CACHE_DURATION = 10 * 60 * 1000`)
- Nombres de ciudades en minúsculas para normalización
- Persistencia automática en localStorage (a través de useLocalStorage)
- Prevención de múltiples cargas simultáneas de la misma ciudad

**Inicialización**:
```typescript
// Ciudad por defecto al iniciar la app
const [cities, setCities] = useLocalStorage<string[]>('cities', ['Ciudad de México']);
```

### 2. useLocalStorage Hook
**Archivo**: `src/hooks/useLocalStorage.ts`

**Propósito**: Sincronización automática entre el estado de React y localStorage.

```typescript
function useLocalStorage<T>(key: string, initialValue: T): 
  [T, (value: T | ((val: T) => T)) => void]
```

**Funcionamiento**:
- Lee del localStorage al montar el componente
- Guarda automáticamente en localStorage cada vez que cambia el estado
- Manejo de errores JSON.parse/stringify

---

## 🎨 Context API

### ThemeContext
**Archivo**: `src/contexts/ThemeContext.tsx`

**Propósito**: Gestión global del tema claro/oscuro.

```typescript
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}
```

**Persistencia**: El tema seleccionado se guarda en `localStorage.theme`

**Uso en la App**:
```typescript
// En App.tsx
const { theme } = useContext(ThemeContext);
const muiTheme = theme === 'light' ? lightTheme : darkTheme;
```

**Temas Disponibles** (`src/services/theme.ts`):
- `lightTheme` - Tema claro con fondo #f4f6f8
- `darkTheme` - Tema oscuro con fondo #121212

---

## 🗺️ Estructura de Páginas (Routing)

### Configuración de Rutas
**Archivo**: `src/App.tsx`

```typescript
<Routes>
  <Route path="/" element={<DashboardPage />} />
  <Route path="/settings" element={<SettingsPage />} />
  <Route path="/map" element={<MapPage />} />
</Routes>
```

### 1. DashboardPage (/)
**Archivo**: `src/pages/DashboardPage.tsx`

**Propósito**: Página principal que muestra tarjetas de clima para todas las ciudades guardadas.

**Características**:
- Renderiza una `WeatherCard` por cada ciudad
- Muestra un estado vacío elegante si no hay ciudades
- Maneja estados de loading y error
- Permite eliminar y actualizar ciudades

### 2. MapPage (/map)
**Archivo**: `src/pages/MapPage.tsx`

**Propósito**: Vista de mapa interactivo con todas las ciudades.

**Características**:
- Mapa Leaflet con OpenStreetMap como base
- Capa de nubes de OpenWeather
- Marcadores personalizados con temperatura
- Popups con preview del clima
- Círculos de área coloreados por temperatura:
  - < 10°C: Azul (#4fc3f7)
  - 10-20°C: Verde (#66bb6a)
  - 20-30°C: Naranja (#ffb74d)
  - > 30°C: Rojo (#ef5350)
- Auto-ajuste de zoom para mostrar todas las ciudades

### 3. SettingsPage (/settings)
**Archivo**: `src/pages/SettingsPage.tsx`

**Propósito**: Configuración y gestión de la aplicación.

**Características**:
- Muestra información de almacenamiento local
- Contador de ciudades guardadas
- Tamaño de datos en localStorage (KB)
- Botón para eliminar todas las ciudades (con confirmación)

---

## 🧩 Componentes Principales

### WeatherCard
**Archivo**: `src/components/WeatherCard/WeatherCard.tsx`

**Props**:
```typescript
{
  weatherData: CombinedWeatherData;
  onRemove: (city: string) => void;
  onRefresh: (city: string) => void;
}
```

**Funcionalidad**:
- Header con gradiente dinámico según condición climática
- Temperatura grande con icono de OpenWeather
- Sensación térmica y temperaturas min/max
- Grid de información: viento, humedad, calidad del aire, amanecer, atardecer, presión
- Recomendaciones contextuales según el clima
- Pronóstico extendido (componente Forecast)
- Mapa opcional (componente WeatherMap)
- Botones: expandir/contraer, mapa, actualizar, eliminar

**Gradientes por Condición Climática**:
- 200-299 (Tormentas): Purple gradient
- 300-599 (Lluvia): Blue-purple gradient
- 600-699 (Nieve): Light gray gradient
- 700-799 (Niebla): Dark gray gradient
- 800 (Despejado): Pink-red gradient
- 801+ (Nublado): Blue gradient

### SearchBar
**Archivo**: `src/components/SearchBar/SearchBar.tsx`

**Props**:
```typescript
{
  onSearch: (city: string) => void;
  recentCities: string[];
}
```

**Funcionalidad**:
- Búsqueda de ciudades con autocompletado
- Muestra ciudades recientes en dropdown
- Validación de entrada
- Responsive (se adapta a móvil)

### Forecast
**Archivo**: `src/components/Forecast/Forecast.tsx`

**Props**:
```typescript
{
  forecast: ForecastData;
}
```

**Funcionalidad**:
- Muestra pronóstico de 5 días
- Agrupa datos por día
- Temperaturas máximas y mínimas diarias
- Iconos de condición climática
- Scroll horizontal en móvil

### Header
**Archivo**: `src/components/layout/Header.tsx`

**Props**:
```typescript
{
  onSearch: (city: string) => void;
  cities: string[];
  weatherDataMap: Map<string, CombinedWeatherData>;
  onUseCurrentLocation?: () => void;
}
```

**Contenido**:
- Logo y título "Clima"
- SearchBar integrado
- Botones de navegación: Dashboard, Mapa, Settings
- Toggle de tema claro/oscuro
- Menú de ciudades (CityListMenu)
- Menú hamburguesa en móvil

### Footer
**Archivo**: `src/components/layout/Footer.tsx`

**Contenido**:
- Copyright
- Links: Sobre nosotros, Contacto, Privacidad
- Íconos de redes sociales (GitHub, Twitter, LinkedIn)

---

## 🛠️ Utilidades

### formatters.ts
**Archivo**: `src/utils/formatters.ts`

**Funciones**:
```typescript
// Formatea timestamp Unix a hora local (HH:mm)
formatTime(timestamp: number, timezoneOffset: number): string

// Formatea timestamp a día de la semana en español
formatDayOfWeek(timestamp: number, timezoneOffset: number): string

// Capitaliza la primera letra de cada palabra
capitalizeDescription(description: string): string

// Obtiene texto y color según AQI (1-5)
getAqiText(aqi: 1|2|3|4|5): { text: string; color: string }

// Formatea temperatura con símbolo de grado
formatTemperature(temp: number): string
```

### recommendations.ts
**Archivo**: `src/utils/recommendations.ts`

**Función Principal**:
```typescript
getWeatherRecommendation(weatherId: number): string
```

**Rangos de Weather ID de OpenWeather**:
- 200-299: Tormentas eléctricas
- 300-399: Llovizna
- 500-599: Lluvia
- 600-699: Nieve
- 700-799: Niebla/Bruma/Humo
- 800: Despejado
- 801-804: Nublado

**Retorna**: Recomendación en español (ej: "¡Está lloviendo! Asegúrate de llevar paraguas.")

---

## 💾 Almacenamiento Local (localStorage)

### Claves Utilizadas:

1. **`cities`** (Array de strings)
   ```json
   ["Ciudad de México", "Monterrey", "Guadalajara"]
   ```
   - Lista de ciudades guardadas por el usuario
   - Usado por: useWeatherData hook

2. **`theme`** (string: "light" | "dark")
   ```json
   "dark"
   ```
   - Preferencia de tema del usuario
   - Usado por: ThemeContext

### Consideraciones para el Backend:
Si se implementa un backend con autenticación, estos datos deberían migrarse a:
- **Base de datos**: Lista de ciudades favoritas por usuario
- **Preferencias de usuario**: Tema y otras configuraciones

---

## 🔄 Flujo de Datos

### 1. Flujo de Búsqueda de Ciudad

```
Usuario escribe ciudad en SearchBar
    ↓
SearchBar.onSearch() → Header.onSearch() → App.weatherHookData.addCity()
    ↓
useWeatherData.addCity(city)
    ↓
Verifica si ya existe (normaliza a minúsculas)
    ↓
Si es nueva:
  - Añade a cities array (se guarda en localStorage)
  - Llama a fetchWeatherData(city)
    ↓
fetchWeatherData() llama a API OpenWeather:
  1. getCoordsByCity(`${city},MX`)
  2. Parallel: getCurrentWeather + get5DayForecast + getAirPollution
    ↓
Actualiza state.data con CombinedWeatherData
    ↓
Componentes se re-renderizan con nuevos datos
```

### 2. Flujo de Renderizado

```
App.tsx
  ├─ Obtiene datos de useWeatherData
  ├─ Pasa props a Header (onSearch, cities, weatherDataMap)
  └─ Router renderiza página actual:
     
     DashboardPage
       ├─ Recibe weatherHook como prop
       ├─ Itera sobre cities array
       └─ Renderiza WeatherCard por cada ciudad
           ├─ Muestra clima actual
           ├─ Forecast component (pronóstico 5 días)
           └─ Opcional: WeatherMap component
     
     MapPage
       ├─ Recibe weatherHook como prop
       ├─ Obtiene citiesWithData de weatherHook.data
       └─ Renderiza mapa Leaflet con marcadores
     
     SettingsPage
       ├─ Recibe weatherHook como prop
       └─ Muestra info y permite borrar ciudades
```

### 3. Flujo de Actualización

```
Usuario hace clic en botón "Actualizar"
    ↓
WeatherCard.onRefresh(city)
    ↓
useWeatherData.refreshCity(city)
    ↓
Ignora cache y hace fetch fresco
    ↓
Actualiza timestamp y datos en state.data
    ↓
Componentes se re-renderizan con datos nuevos
```

---

## 🌐 Consideraciones para Desarrollo de Backend

### Endpoints Sugeridos para API Backend:

Si decides reemplazar OpenWeather con un backend propio, aquí están los endpoints mínimos necesarios:

#### 1. **GET /api/weather/search**
Búsqueda y datos de clima para una ciudad.

**Query Params**:
- `city` (string, required): Nombre de la ciudad
- `country` (string, optional): Código de país (default: "MX")

**Response**:
```json
{
  "location": {
    "name": "Ciudad de México",
    "lat": 19.4326,
    "lon": -99.1332,
    "country": "MX",
    "state": "Ciudad de México"
  },
  "currentWeather": {
    "temp": 18.5,
    "feels_like": 17.2,
    "temp_min": 15.0,
    "temp_max": 22.0,
    "humidity": 65,
    "pressure": 1013,
    "visibility": 10000,
    "wind": { "speed": 3.5, "deg": 180 },
    "clouds": { "all": 40 },
    "weather": [{
      "id": 801,
      "main": "Clouds",
      "description": "algo de nubes",
      "icon": "02d"
    }],
    "sunrise": 1638345600,
    "sunset": 1638388800,
    "timezone": -21600,
    "dt": 1638360000
  },
  "forecast": {
    "list": [
      {
        "dt": 1638363600,
        "dt_txt": "2024-11-23 15:00:00",
        "main": {
          "temp": 19.0,
          "feels_like": 18.0,
          "temp_min": 18.0,
          "temp_max": 20.0,
          "humidity": 60
        },
        "weather": [{ "id": 800, "main": "Clear", "description": "cielo claro", "icon": "01d" }],
        "clouds": { "all": 0 },
        "wind": { "speed": 2.5 },
        "pop": 0.0
      }
      // ... más items cada 3 horas
    ],
    "city": {
      "name": "Ciudad de México",
      "country": "MX",
      "sunrise": 1638345600,
      "sunset": 1638388800,
      "timezone": -21600
    }
  },
  "airPollution": {
    "list": [{
      "main": { "aqi": 2 }
    }]
  }
}
```

#### 2. **GET /api/weather/cities** (Autenticado)
Obtener ciudades favoritas del usuario.

**Headers**:
- `Authorization: Bearer <token>`

**Response**:
```json
{
  "cities": ["Ciudad de México", "Monterrey", "Guadalajara"]
}
```

#### 3. **POST /api/weather/cities** (Autenticado)
Añadir ciudad a favoritos.

**Headers**:
- `Authorization: Bearer <token>`

**Body**:
```json
{
  "city": "Cancún"
}
```

**Response**:
```json
{
  "success": true,
  "cities": ["Ciudad de México", "Monterrey", "Guadalajara", "Cancún"]
}
```

#### 4. **DELETE /api/weather/cities/:city** (Autenticado)
Eliminar ciudad de favoritos.

**Headers**:
- `Authorization: Bearer <token>`

**Response**:
```json
{
  "success": true,
  "cities": ["Ciudad de México", "Monterrey"]
}
```

#### 5. **GET /api/user/preferences** (Autenticado)
Obtener preferencias del usuario (tema, etc.)

**Response**:
```json
{
  "theme": "dark",
  "language": "es"
}
```

#### 6. **PUT /api/user/preferences** (Autenticado)
Actualizar preferencias.

**Body**:
```json
{
  "theme": "light"
}
```

---

## 🔐 Autenticación (Para Backend Futuro)

### Flujo Sugerido:

1. **Login/Register**
   ```typescript
   POST /api/auth/login
   POST /api/auth/register
   ```
   - Retorna JWT token
   - Frontend guarda token en localStorage o httpOnly cookie

2. **Modificación de Axios Client**
   ```typescript
   // En src/api/backend.ts (nuevo archivo)
   const apiClient = axios.create({
     baseURL: process.env.REACT_APP_API_URL,
   });

   // Interceptor para añadir token
   apiClient.interceptors.request.use((config) => {
     const token = localStorage.getItem('authToken');
     if (token) {
       config.headers.Authorization = `Bearer ${token}`;
     }
     return config;
   });
   ```

3. **Context de Autenticación**
   ```typescript
   // Nuevo archivo: src/contexts/AuthContext.tsx
   interface AuthContextType {
     user: User | null;
     login: (email: string, password: string) => Promise<void>;
     logout: () => void;
     isAuthenticated: boolean;
   }
   ```

---

## 📦 Scripts Disponibles

```bash
# Desarrollo local (puerto 3000)
npm start

# Build para producción
npm run build

# Tests unitarios
npm test

# Deploy a GitHub Pages
npm run deploy

# Eject configuración (NO RECOMENDADO)
npm run eject
```

---

## 🧪 Testing

### Estado Actual:
- Configurado con React Testing Library
- Archivo de setup: `src/setupTests.ts`
- No hay tests implementados actualmente

### Sugerencias para Backend:
Si añades tests, considera:
- Mock de llamadas a API externa (OpenWeather)
- Tests de integración para flujo completo de búsqueda
- Tests de hooks personalizados (useWeatherData, useLocalStorage)
- Tests de componentes con diferentes estados (loading, error, success)

---

## 🎨 Estilos y Diseño

### Sistema de Diseño:
- **Colores primarios**: Basados en Material-UI
  - Light theme: #1976d2 (azul)
  - Dark theme: #90caf9 (azul claro)
- **Gradientes**: Dinámicos según condición climática
- **Espaciado**: Sistema de spacing de MUI (múltiplos de 8px)
- **Responsive breakpoints**:
  - xs: 0px
  - sm: 600px
  - md: 900px
  - lg: 1200px
  - xl: 1536px

### CSS Modules:
Varios componentes usan CSS Modules (*.module.css) para estilos aislados.

---

## 🚀 Deploy y CI/CD

### GitHub Actions:
**Archivo**: `.github/workflows/deploy.yml`

**Trigger**: Push a rama `main`

**Pasos**:
1. Checkout del código
2. Setup Node.js
3. npm install
4. npm run build
5. Deploy a GitHub Pages

### Variables de Entorno:
Para backend futuro, considera usar:
```env
REACT_APP_API_URL=https://api.tudominio.com
REACT_APP_API_KEY=your-api-key
```

---

## 📝 Notas para el Equipo de Backend

### 1. **Compatibilidad de Datos**
Los tipos TypeScript en `src/api/types.ts` deben coincidir **exactamente** con los que retorne tu backend.

### 2. **CORS**
Tu backend debe permitir peticiones desde:
- `http://localhost:3000` (desarrollo)
- `https://rafaello129.github.io` (producción)

### 3. **Rate Limiting**
Considera implementar rate limiting para proteger tu API.

### 4. **Caché**
El frontend cachea datos por 10 minutos. Tu backend podría:
- Implementar caché con Redis
- Retornar headers de caché (`Cache-Control`)
- Usar ETags para validación

### 5. **Errores**
Usa códigos HTTP estándar:
- 200: Success
- 400: Bad Request (ciudad inválida)
- 401: No autenticado
- 403: No autorizado
- 404: Ciudad no encontrada
- 500: Error del servidor

### 6. **Locale**
Actualmente todo está en español. Si añades multi-idioma:
- Accept-Language header
- Query param `?lang=es`
- Campo en preferencias de usuario

### 7. **Limitación Actual**
La búsqueda está hardcodeada a México (`,MX`):
```typescript
const location = await getCoordsByCity(`${city},MX`);
```
Considera hacerlo configurable para otros países.

---

## 🔗 URLs y Enlaces Importantes

- **Repositorio**: https://github.com/rafaello129/app-clima-Rafael-Ramos
- **App en vivo**: https://rafaello129.github.io/clima
- **OpenWeather API**: https://openweathermap.org/api
- **Material-UI Docs**: https://mui.com/
- **React Router**: https://reactrouter.com/
- **Leaflet**: https://leafletjs.com/

---

## 📞 Puntos de Contacto para Integración

Si el equipo de backend necesita clarificaciones, estos son los archivos clave:

1. **Tipos de datos**: `src/api/types.ts`
2. **Cliente API actual**: `src/api/openWeather.ts`
3. **Hook principal**: `src/hooks/useWeatherData.ts`
4. **Ejemplo de uso**: `src/pages/DashboardPage.tsx`

---

## ✅ Checklist para Backend

- [ ] Crear endpoints REST equivalentes a OpenWeather
- [ ] Implementar autenticación JWT
- [ ] Crear modelo de usuario con ciudades favoritas
- [ ] Implementar caché de datos meteorológicos
- [ ] Configurar CORS para el dominio del frontend
- [ ] Rate limiting por usuario/IP
- [ ] Documentar API con Swagger/OpenAPI
- [ ] Tests de integración
- [ ] Monitoreo y logging
- [ ] Deploy en servidor (Heroku, AWS, etc.)

---

**Última actualización**: 2025-11-23
**Versión del proyecto**: 0.1.0
**Autor de la documentación**: GitHub Copilot Agent
