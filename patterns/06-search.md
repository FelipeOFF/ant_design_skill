# Pattern 06: Search

## Problem / Context

Search interfaces need to balance responsiveness with server load. Debouncing prevents excessive API calls while providing real-time feedback to users.

## When to Use

- Global search across entities
- Filterable lists and tables
- Autocomplete/typeahead fields
- Search-as-you-type experiences

## When NOT to Use

- For exact-match lookups (use direct query)
- When search results are computable client-side
- For sensitive data requiring explicit submit

## AntD Components Involved

- `Input.Search` - Search input with button
- `AutoComplete` - Typeahead suggestions
- `List` or `Table` - Results display
- `Spin` - Loading indicator
- `Empty` - No results state
- `Tag` - Applied filters display
- `Space` - Filter chips layout

## React Implementation Notes

### Debounce Implementation

```tsx
import { useState, useEffect, useCallback } from 'react';
import { debounce } from 'lodash-es'; // or use your own

const [searchTerm, setSearchTerm] = useState('');
const [debouncedTerm, setDebouncedTerm] = useState('');

const debouncedUpdate = useCallback(
  debounce((value: string) => setDebouncedTerm(value), 300),
  []
);

useEffect(() => {
  debouncedUpdate(searchTerm);
  return () => debouncedUpdate.cancel();
}, [searchTerm, debouncedUpdate]);
```

### Search with Loading State

```tsx
const [loading, setLoading] = useState(false);
const [results, setResults] = useState<Result[]>([]);
const [hasSearched, setHasSearched] = useState(false);

useEffect(() => {
  if (!debouncedTerm) {
    setResults([]);
    setHasSearched(false);
    return;
  }

  const fetchResults = async () => {
    setLoading(true);
    try {
      const data = await api.search(debouncedTerm);
      setResults(data);
      setHasSearched(true);
    } finally {
      setLoading(false);
    }
  };

  fetchResults();
}, [debouncedTerm]);
```

### AbortController for Race Conditions

```tsx
useEffect(() => {
  const controller = new AbortController();
  
  const fetchResults = async () => {
    try {
      const data = await api.search(debouncedTerm, { signal: controller.signal });
      setResults(data);
    } catch (error) {
      if (error.name !== 'AbortError') {
        handleError(error);
      }
    }
  };

  if (debouncedTerm) fetchResults();
  
  return () => controller.abort();
}, [debouncedTerm]);
```

## Accessibility / Keyboard

- Search input must have an accessible label
- Results should be announced via live region
- Arrow keys should navigate through suggestions
- Enter should submit the search
- Clear button must be keyboard accessible

## Do / Don't

| Do | Don't |
|----|-------|
| Debounce at 300-500ms | Search on every keystroke |
| Show loading indicator | Leave user waiting without feedback |
| Handle empty results gracefully | Show "undefined" or blank |
| Cancel in-flight requests | Show stale results |
| Trim whitespace from search | Search for "   " |
| Persist recent searches (optional) | Forget user's previous searches |

## Minimal Code Snippet

```tsx
import { useState, useEffect, useCallback } from 'react';
import { Input, List, Spin, Empty, Tag, Space } from 'antd';
import { SearchOutlined, CloseCircleOutlined } from '@ant-design/icons';
import { debounce } from 'lodash-es';

interface SearchResult {
  id: string;
  title: string;
  description: string;
  category: string;
}

export function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedTerm, setDebouncedTerm] = useState('');
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState<SearchResult[]>([]);
  const [hasSearched, setHasSearched] = useState(false);

  // Debounce search term
  const debouncedUpdate = useCallback(
    debounce((value: string) => setDebouncedTerm(value), 300),
    []
  );

  useEffect(() => {
    debouncedUpdate(searchTerm);
    return () => debouncedUpdate.cancel();
  }, [searchTerm, debouncedUpdate]);

  // Fetch results when debounced term changes
  useEffect(() => {
    const controller = new AbortController();

    const fetchResults = async () => {
      if (!debouncedTerm.trim()) {
        setResults([]);
        setHasSearched(false);
        return;
      }

      setLoading(true);
      try {
        // Simulate API call
        await new Promise((resolve) => setTimeout(resolve, 500));
        
        const mockResults: SearchResult[] = Array.from({ length: 5 }, (_, i) => ({
          id: `${i}`,
          title: `${debouncedTerm} - Result ${i + 1}`,
          description: `Description for search result ${i + 1} matching "${debouncedTerm}"`,
          category: ['Article', 'User', 'Product'][i % 3],
        }));

        if (!controller.signal.aborted) {
          setResults(mockResults);
          setHasSearched(true);
        }
      } finally {
        if (!controller.signal.aborted) {
          setLoading(false);
        }
      }
    };

    fetchResults();
    return () => controller.abort();
  }, [debouncedTerm]);

  const handleClear = () => {
    setSearchTerm('');
    setResults([]);
    setHasSearched(false);
  };

  return (
    <div style={{ maxWidth: 600, margin: '0 auto', padding: 24 }}>
      <Input.Search
        placeholder="Search articles, users, products..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        enterButton={<SearchOutlined />}
        size="large"
        loading={loading}
        allowClear
        aria-label="Search"
      />

      <div style={{ marginTop: 24 }}>
        {loading && (
          <div style={{ textAlign: 'center', padding: 40 }}>
            <Spin size="large" />
          </div>
        )}

        {!loading && hasSearched && results.length === 0 && (
          <Empty
            image={Empty.PRESENTED_IMAGE_SIMPLE}
            description={
              <span>
                No results for "<strong>{debouncedTerm}</strong>"
              </span>
            }
          />
        )}

        {!loading && results.length > 0 && (
          <List
            header={
              <Space>
                <span>{results.length} results for "{debouncedTerm}"</span>
                <Tag closable onClose={handleClear}>Clear</Tag>
              </Space>
            }
            dataSource={results}
            renderItem={(item) => (
              <List.Item>
                <List.Item.Meta
                  title={item.title}
                  description={
                    <div>
                      <Tag size="small">{item.category}</Tag>
                      <p>{item.description}</p>
                    </div>
                  }
                />
              </List.Item>
            )}
          />
        )}
      </div>
    </div>
  );
}
```
