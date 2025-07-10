---
layout: post
title: Manage your configuration of all CLI tools with GNU Stow
date: 2025-08-11
---
בפוסט זה אלמד כיצד לשמור ולנהל במקום אחד את כל הקונפיגורציה של כלי ה- CLI שלכם
באמצעות **GNU Stow**.

## מבוא

כל תוכנות ה- `CLI` יש להם קבצי הגדרות בפורמטים שונים (`yaml`, `toml`,
`plain text`, ``sh``, `zsh`, etc.) והם פזורים במיקומים שונים במערכת הקבצים.
המטרה של הפוסט הזה הוא להשיג שני מטרות:

1. ניהול כל קבצי התצורה במקום אחד וקבוע,
2. שימוש ב- VCS (Version Control System) (שזה Git) כדי להשיג שני אפקטים:
  a) שמירה על גרסאות על מנת לחזור לגירסה קודמת (Rollback) במידת הצורך,
  b) לפבלש (To publish it) ב- Github.

### מה זה GNU Stow

ה- GNU Stow הוא כלי CLI שיוצר Symlinks בין קבצי קונפיגורציה במקום אחד למיקומם
המקורי במערכת הקבצים כדי לאפשר לנו המשתמשים למצוא את כל הקבצים מסודרים במקום
אחד מסודרים לפי כלים (למשל תיקייה עבור `zsh`, ותיקייה עבור `vim`)  ומאידך
לאפשר לכל התוכנות למצוא את קבצי התצורה שלהן במיקום המוגדר להן. להלן הסבר.

### מיקום קבצי הקונפיגורציה

כל CLI Tool מוגדר לו לחפש קבצי הגדרות במיקומים ספציפיים (לעיתים אפשרות אחד
ולעיתים כמה) כצעד ראשון לאחר הרצת התוכנה. להלן טבלה עם מספר דוגמאות:

| CLI Tool  | Original location config file        |
| --------- | ------------------------------------ |
| Zsh       | `~/.zshrc`                           |
| Alacritty | `~/.config/alacritty/alacritty.toml` |
| Wezterm   | `~/.config/wezterm/wezterm.lua`      |
| Kitty     | `~/.config/kitty/kitty.conf`         |

### יצירת לינקים

כעת נראה את התוצאה הסופית, (כאשר התהליך יוסבר מיד), כל קבצי הקונפיגורציה
נמצאים בתיקייה אחת ומשם נוצרים קבצי לינקים במיקומים המקוריים שהתוכנה מחפשת;
ובעצם היא הולכת לתיקייה שאנו מפנים את הלינק:

```sh
❯ tree ~/dotfiles

/home/nachmen/dotfiles
├── alacritty
│   └── .config
│       └── alacritty
│           └── alacritty.toml
├── kitty
│   └── .config
│       └── kitty
│           ├── current-theme.conf
│           └── kitty.conf
├── wezterm
│   └── .config
│       └── wezterm
│           └── wezterm.lua
└── zsh
    ├── .aliases.zsh
    ├── .p10k.zsh
    ├── .zimrc
    ├── .zshenv
    └── .zshrc
```

ואם נסתכל על קובץ הקונפיגורציה של Allacritty לדוגמה:

```sh
❯ ls -l ~/.config/alacritty/

alacritty.toml -> ../../dotfiles/alacritty/.config/alacritty/alacritty.toml
```

שימו לב, הקובץ הוא מפנה לקובץ שבתיקיית **dotfiles**

### הפרקטיקה שלב אחר שלב

כעת נעבור בזריזות על סדר הפעולות כדי להגיע לתוצאה שהוצגה

#### יצירת תיקיית **dotfiles**

בתיקיית הבית צרו תיקייה חדשה עבור כל קבצי הקונפיגורציה, ולמען הבהירות
קראו לה *dotfiles*:

```sh
❯ mkdir ~/dotfiles
```

#### צרו את מבנה התיקיות עבור הקבצים של Zsh

כעת עבור כל תוכנה ניצור תיקייה, להלן אדגים על ארבעת התוכנות הבאות:
Zsh (מעטפת שורת פקודה מתקדמת), Alacritty (טרמינל מהיר מאוד),
Kitty (טרמינל מהיר ויפה), Wezterm (עוד טרמינל...).

נתחיל עם קבצי תצורת מעטפת Zsh:

```sh
❯ mkdir -p ~/dotfiles/zsh/
```

#### העברת הקבצים המקוריים

בתוך התיקייה החדשה נעביר את קובץ התצורה של Zsh:

```sh
❯ mv ~/.zshrc ~/.aliases.zsh ~/.p10k.zsh ~/.zimrc ~/.zshenv ~/dotfiles/zsh
```

נסביר: העברנו חמישה קבצים מתיקיית הבית אל התיקייה החדשה. כעת נריץ את
הפקודה `stow` כדי ליצור את הלינקים לתיקיית הבית:

```sh
# go to th 'dotfiles' directory
❯ cd ~/dotfiles

# make symlinks of 'zsh'
❯ stow zsh
```

כעת נראה שנוצרו לינקים במקום הקבצים המקוריים.

```sh
❯ ls --oneline \
~/.zshrc \
~/.aliases.zsh \
~/.zshenv \
~/.p10k.zsh \
~/.p10k.zsh

/home/nachmen/.aliases.zsh -> dotfiles/zsh/.aliases.zsh
/home/nachmen/.p10k.zsh -> dotfiles/zsh/.p10k.zsh
/home/nachmen/.p10k.zsh -> dotfiles/zsh/.p10k.zsh
/home/nachmen/.zshenv -> dotfiles/zsh/.zshenv
/home/nachmen/.zshrc -> dotfiles/zsh/.zshrc
```

זה העיקרון. כך תוכלו לעשות עבור כל תוכנה שברצונכם לשמור ולנהל את קבצי
התצורה שלהן במקום אחד.
