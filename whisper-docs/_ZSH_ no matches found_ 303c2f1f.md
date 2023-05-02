The shell is interpreting the `?` in the URL as a globbing wildcard, so because of this it tries to find files matching the pattern [`https://www.youtube.com/watch?`](https://www.youtube.com/watch?), which of course it will fail to find. To prevent the shell from interpreting the `?` or other characters like `*` and `^` as wildcards, you may:

1. Escape it with a backslash: `\?`
2. Wrap the URL in quotes: `'`[`https://www.youtube.com/watch?v=mVEwurFdnYk'`](https://www.youtube.com/watch?v=mVEwurFdnYk')
3. Use the `noglob` precommand modifier like [/u/grumpycrash](https://www.reddit.com/u/grumpycrash/) suggested.

In general, it's a good habit to quote arguments to prevent unexpected interpretation by the shell.
