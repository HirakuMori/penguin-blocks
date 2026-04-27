# Penguin Blocks

Browser game repository for hosting multiple mini games via GitHub Pages.

## Repository Layout

- `docs/`: GitHub Pages deployment root.
  - `docs/index.html`: Top page with game list.
  - `docs/games/<game-name>/`: Each playable game.
- `design-docs/`: Non-deployed planning and game design documents.

## Play

After deploying GitHub Pages, open:

- Top page: `/`
- Penguin Blocks: `/games/penguin-blocks/`
- 21 or Curse: `/games/21-or-curse/`

## GitHub Pages

1. Open the repository on GitHub.
2. Go to **Settings** > **Pages**.
3. Set **Build and deployment** to **Deploy from a branch**.
4. Select the `main` branch and `/docs`.
5. Save the settings.

## Game Design Docs

- [21 or Curse MVP spec](design-docs/21-or-curse-spec.md)
- [21 or Curse prototype](docs/games/21-or-curse/index.html)

## Intellectual Property Note

This is an independent, non-commercial falling-block puzzle project. It is not affiliated with, endorsed by, or sponsored by The Tetris Company or any official Tetris product.

The project avoids using official Tetris assets. The source code in this repository is licensed under the MIT License, but that license only covers this repository's original code and assets.

## License

MIT License. See [LICENSE](LICENSE).
