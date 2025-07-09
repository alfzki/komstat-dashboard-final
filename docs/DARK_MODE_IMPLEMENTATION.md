# Dark Mode Synchronization Implementation Guide

## Overview

This implementation provides seamless dark mode synchronization between the React frontend and embedded R Shiny applications. The system ensures that when users toggle dark mode in the React frontend, all Shiny iframes automatically switch to match the selected theme.

## Architecture

### 1. Components

- **ShinyDarkModeProvider**: Global context provider for centralized dark mode state management
- **useShinyDarkMode**: Custom hook for individual iframe dark mode synchronization
- **Enhanced Shiny JavaScript**: Advanced message handling in R Shiny applications

### 2. Communication Flow

```
React Frontend (Dark Mode Toggle)
    ↓
ShinyDarkModeProvider (Global State)
    ↓
useShinyDarkMode Hook (Individual iframes)
    ↓
postMessage API (Cross-frame communication)
    ↓
Shiny JavaScript Listener (Message handling)
    ↓
CSS Class Toggle (Visual changes)
```

## Implementation Details

### Frontend (React)

#### Global Context Provider
```jsx
// In Dashboard.jsx
<ShinyDarkModeProvider>
  <App />
</ShinyDarkModeProvider>
```

#### Using the Hook
```jsx
// In any component with Shiny iframe
const {
  iframeRef,
  isDarkMode,
  handleIframeLoad,
  isReady,
  usingGlobalContext
} = useShinyDarkMode({
  shinyOrigin: 'http://127.0.0.1:3838',
  enableLogging: true
});
```

#### Iframe Setup
```jsx
<iframe
  ref={iframeRef}
  src="http://127.0.0.1:3838"
  onLoad={handleIframeLoad}
  // ... other props
/>
```

### Backend (R Shiny)

The Shiny application includes enhanced JavaScript that:

1. **Listens for messages** from the React frontend
2. **Validates message origins** for security
3. **Applies dark mode styles** via CSS classes
4. **Sends ready confirmations** back to React
5. **Handles multiple message types** (DARK_MODE, REQUEST_READY)

## Security Features

### Origin Validation
Both React and Shiny validate message origins:

**Trusted Origins:**
- `http://localhost:3000` (React dev server)
- `http://127.0.0.1:3000` (React dev server)
- `http://localhost:5173` (Vite dev server)
- `http://127.0.0.1:5173` (Vite dev server)

### Message Structure
```javascript
{
  type: 'DARK_MODE',
  value: boolean,
  timestamp: number,
  source: string
}
```

## Features

### 1. Automatic Synchronization
- Dark mode changes in React immediately propagate to all Shiny iframes
- No manual configuration required for individual iframes

### 2. Fallback Support
- If global context is unavailable, individual hooks work independently
- Graceful degradation ensures functionality in all scenarios

### 3. Error Handling
- Comprehensive logging for debugging
- Graceful handling of communication failures
- Security checks prevent unauthorized access

### 4. Performance Optimization
- Efficient message batching
- Minimal re-renders through optimized state management
- Lazy loading and delayed initialization

## Usage Examples

### Basic Usage
```jsx
import useShinyDarkMode from '../hooks/useShinyDarkMode';

const MyShinyComponent = () => {
  const { iframeRef, handleIframeLoad } = useShinyDarkMode();
  
  return (
    <iframe
      ref={iframeRef}
      src="http://127.0.0.1:3838"
      onLoad={handleIframeLoad}
    />
  );
};
```

### Advanced Configuration
```jsx
const { 
  iframeRef, 
  isDarkMode, 
  sendDarkModeToShiny,
  isReady 
} = useShinyDarkMode({
  shinyOrigin: 'http://localhost:3838',
  loadDelay: 1000,
  enableLogging: false,
  useGlobalContext: true
});
```

## Troubleshooting

### Common Issues

1. **Messages not received**
   - Check browser console for security warnings
   - Verify origin URLs match exactly
   - Ensure iframe has loaded completely

2. **Dark mode not applying**
   - Verify CSS classes are defined in Shiny
   - Check for JavaScript errors in Shiny console
   - Confirm message structure is correct

3. **Performance issues**
   - Reduce logging in production
   - Optimize CSS transitions
   - Use debouncing for rapid mode changes

### Debugging

Enable detailed logging:
```jsx
useShinyDarkMode({
  enableLogging: true
});
```

Check browser console for:
- Message sending/receiving logs
- Security warnings
- Error messages

## Best Practices

1. **Always validate origins** for security
2. **Use global context** for multiple iframes
3. **Handle errors gracefully** with fallbacks
4. **Test across different browsers** and devices
5. **Minimize logging** in production builds

## File Structure

```
frontend/
├── src/
│   ├── contexts/
│   │   └── ShinyDarkModeContext.jsx
│   ├── hooks/
│   │   └── useShinyDarkMode.js
│   ├── components/
│   │   └── views/
│   │       └── AnalisisNonParametrikView.jsx
│   └── Dashboard.jsx
server/
└── app/
    └── app.R (Enhanced JavaScript)
```

## Future Enhancements

1. **Theme customization** beyond dark/light
2. **Multiple Shiny app support** with different configurations
3. **Offline mode** detection and handling
4. **Performance metrics** and monitoring
5. **Theme persistence** across sessions

## Testing

### Manual Testing
1. Toggle dark mode in React frontend
2. Verify immediate changes in Shiny iframe
3. Test with multiple iframes
4. Check browser console for errors

### Automated Testing
- Unit tests for hooks and contexts
- Integration tests for message passing
- E2E tests for full user workflows

## Configuration

### Environment Variables
```bash
REACT_APP_SHINY_ORIGIN=http://127.0.0.1:3838
REACT_APP_ENABLE_DARK_MODE_LOGGING=true
```

### Build Configuration
Ensure proper build settings for cross-origin communication in production environments.
