# Pattern 6: Debounced Search & Filtering

## Problem / Context
Live search functionality that fetches results from a server without spamming API requests for every keystroke.

## When to use
- Global search bars.
- Autocomplete fields (Select with search).
- Table filter inputs.

## When NOT to use
- Client-side filtering of small lists (just filter the array directly).

## AntD Components
- `Input.Search`
- `Select` (with `showSearch`)
- `AutoComplete`

## React Implementation Notes
- **Debounce**: Use `lodash.debounce` or a custom `useDebounce` hook.
- **Effect**: Trigger the API call in `useEffect` dependent on the debounced value.
- **Cleanup**: If using `lodash.debounce`, remember to `cancel()` it on unmount.

## Accessibility / Keyboard
- Use `aria-label="Search"` on the input.
- Ensure the results list (if custom) manages focus or uses `aria-activedescendant`. AntD `Select`/`AutoComplete` handles this.

## Do / Don't
- **Do**: Show a loading spinner (`loading` prop) while fetching.
- **Do**: Handle empty states or "No results found".
- **Don't**: Debounce too aggressively (e.g., > 1s) causing laggy feel, or too little (< 200ms) causing load. 300-500ms is the sweet spot.

## Minimal Code Snippet

```tsx
import React, { useState, useMemo, useRef } from 'react';
import { Select, Spin } from 'antd';
import type { SelectProps } from 'antd';
import debounce from 'lodash/debounce'; // assume lodash is installed

interface UserValue {
  label: string;
  value: string;
}

export const DebouncedUserSearch: React.FC = () => {
  const [options, setOptions] = useState<UserValue[]>([]);
  const [fetching, setFetching] = useState(false);

  const fetchUserList = async (username: string) => {
    if (!username) {
        setOptions([]);
        return;
    }
    setFetching(true);
    // Simulate API
    setTimeout(() => {
        setOptions([
            { label: `${username} 1`, value: `${username}1` },
            { label: `${username} 2`, value: `${username}2` },
        ]);
        setFetching(false);
    }, 600);
  };

  const debounceFetcher = useMemo(() => {
    const loadOptions = (value: string) => {
      setOptions([]);
      setFetching(true);
      fetchUserList(value);
    };
    return debounce(loadOptions, 500);
  }, []);

  return (
    <Select
      showSearch
      labelInValue
      filterOption={false}
      onSearch={debounceFetcher}
      notFoundContent={fetching ? <Spin size="small" /> : null}
      options={options}
      placeholder="Search users..."
      style={{ width: '100%' }}
    />
  );
};
```
