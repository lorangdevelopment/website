<script lang="ts" module>
    import { derived, writable } from 'svelte/store';

    export type Theme = 'dark' | 'light' | 'system';
    export const currentTheme = (function () {
        const store = writable<Theme>(getPreferredTheme());

        const set: typeof store.set = (value) => {
            store.set(value);
            if (browser) {
                localStorage.setItem('theme', value);
                document.documentElement.style.setProperty('color-scheme', value);
            }
        };

        return { ...store, set };
    })();

    export const themeInUse = derived(currentTheme, (theme) => {
        return theme === 'system' ? getSystemTheme() : theme;
    });

    function isTheme(theme: unknown): theme is Theme {
        return ['dark', 'light', 'system'].includes(theme as Theme);
    }

    function getSystemTheme(): Theme {
        return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
    }

    function getPreferredTheme() {
        if (!browser) {
            return 'dark';
        }

        const theme = localStorage.getItem('theme');

        if (!isTheme(theme)) {
            return 'dark';
        }

        return theme;
    }
</script>

<script lang="ts">
	import '../app.css';
	import { browser } from '$app/environment';
	let { children } = $props();

	function applyTheme(theme: Theme) {
        const resolvedTheme = theme === 'system' ? getSystemTheme() : theme;
        const className = `${resolvedTheme}`;
        document.body.classList.remove('dark', 'light');
        document.body.classList.add(className);
    }

	$effect(() => {
		const initialTheme = getSystemTheme();
		applyTheme(initialTheme);
	})
</script>

{@render children()}
