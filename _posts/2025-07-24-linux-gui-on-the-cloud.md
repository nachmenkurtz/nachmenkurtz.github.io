---
layout: post
title: Run Linux GUI on the AWS cloud
---

מדריך זה מסביר כיצד להתקין סביבת שולחן עבודה גרפית מלאה (GNOME) על שרת EC2 עם **Amazon Linux 2023** ולהתחבר אליה באופן מאובטח באמצעות VNC על גבי מנהרת SSH. המדריך כולל גם הוראות לניהול והוספת משתמשים.

---

## שלב 1: הקמת והגדרת שרת EC2

תחילה, יש להקים שרת EC2 חדש בחשבון ה-AWS שלכם.

1. **הקמת השרת:**

      * **AMI:** בחרו `Amazon Linux 2023 AMI`.
      * **Instance Type:** מומלץ לבחור לפחות `t3.medium` (עם 4GB RAM) כדי להבטיח ביצועים חלקים של הסביבה הגרפית.
      * **Key Pair:** ודאו שאתם בוחרים או יוצרים זוג מפתחות גישה (key pair) ושומרים את קובץ ה-`.pem` במקום בטוח.

2. **הגדרת Security Group (חוקי אבטחה):**
    זהו שלב קריטי לאבטחה. אנו נפתח **רק** את הפורט הנדרש לגישת SSH. החיבור הגרפי יעבור דרך מנהרה מאובטחת על גבי חיבור זה.

      * צרו חוק **Inbound** חדש עם ההגדרות הבאות:
          * **Type:** `SSH`
          * **Protocol:** `TCP`
          * **Port Range:** `22`
          * **Source:** `My IP` (כדי לאפשר גישה רק מכתובת ה-IP הנוכחית שלכם).

    ⚠️ **שימו לב:** **לעולם אין לפתוח את פורט 5901** (או פורטי VNC אחרים) באופן ישיר\! פתיחת הפורט חושפת את השרת להתקפות ומשדרת את התעבורה הגרפית ללא הצפנה.

-----

## שלב 2: התקנת סביבת GNOME ושרת VNC

לאחר שהשרת רץ, התחברו אליו באמצעות SSH.

```bash
# החליפו את 'your-key.pem' ואת 'public-ip-address' בפרטים שלכם
ssh -i "your-key.pem" ec2-user@public-ip-address
```

כעת, הריצו את הפקודות הבאות בטרמינל של השרת:

1. **עדכון המערכת:**

    ```bash
    sudo dnf update -y
    ```

2. **התקנת סביבת העבודה הגרפית (GNOME):**
    הפקודה הבאה מתקינה את כל החבילות הנדרשות עבור שולחן העבודה של GNOME. שם הקבוצה הנכון הוא "Desktop".

    ```bash
    sudo dnf groupinstall "Desktop" -y
    ```

3. **התקנת שרת TigerVNC:**
    זוהי תוכנת השרת שתאפשר לנו להתחבר גרפית.

    ```bash
    sudo dnf install -y tigervnc-server
    ```

-----

## שלב 3: הגדרת שרת ה-VNC למשתמש הראשי

1. **הגדרת סיסמת VNC:**
    הריצו את הפקודה הבאה והקלידו סיסמה חזקה. זוהי הסיסמה שתשמש אתכם להתחברות הגרפית.

    ```bash
    vncpasswd
    ```

      * תתבקשו להזין סיסמה ולוודא אותה.
      * כאשר תישאלו אם להגדיר סיסמת "view-only", הקישו `n` (no) ואנטר.

2. **הגדרת משתמש לשירות ה-VNC:**
    יש לקבוע איזה משתמש מערכת יקושר לכל סשן VNC. נערוך את קובץ ההגדרות הראשי:

    ```bash
    sudo vi /etc/tigervnc/vncserver.users
    ```

      * הקישו `i` כדי להיכנס למצב עריכה (insert mode).
      * הוסיפו את השורה הבאה, המקשרת את תצוגה מספר 1 (`:1`) למשתמש `ec2-user`:

        ```bash
        :1=ec2-user
        ```

      * לשמירה ויציאה: לחצו `Esc`, ולאחר מכן הקלידו `:wq` ולחצו אנטר.

3. **הגדרת פרמטרים לסשן ה-VNC:**
    כעת נגדיר את מאפייני ברירת המחדל לסשן, כמו הרזולוציה וסוג שולחן העבודה.

    ```bash
    sudo vi /etc/tigervnc/vncserver-config-defaults
    ```

      * הקישו `i` כדי להיכנס למצב עריכה והדביקו את התוכן הבא בקובץ:

        ```bash
        session=gnome
        securitytypes=vncauth,tlsvnc
        geometry=1920x1080
        localhost
        alwaysshared
        ```

      * שמרו וצאו (`Esc`, ואז `:wq`).

-----

## שלב 4: הפעלת שירות ה-VNC

הפעל את שירות ה-VNC עבור תצוגה מספר 1 (`:1`) והגדר אותו לעלות אוטומטית עם הפעלת השרת.

```bash
sudo systemctl start vncserver@:1
sudo systemctl enable vncserver@:1
```

כדי לוודא שהשירות רץ בהצלחה, השתמשו בפקודה: `sudo systemctl status vncserver@:1`.

-----

## שלב 5: חיבור מאובטח מהמחשב האישי

כדי להתחבר, ניצור "מנהרה" מאובטחת ומוצפנת באמצעות SSH. שלב זה מבוצע **במחשב האישי שלכם**.

#### א. יצירת מנהרת SSH (SSH Tunnel)

* **ב-Windows (באמצעות PuTTY):**

    1. פתחו את PuTTY.
    2. ב-**Session**, הזינו את כתובת ה-IP של השרת.
    3. נווטו אל **Connection -\> SSH -\> Tunnels**.
    4. הגדירו את המנהרה:
          * **Source port:** `5901`
          * **Destination:** `localhost:5901`
          * לחצו על **Add**.
    5. נווטו אל **Connection -\> SSH -\> Auth -\> Credentials** ובחרו את קובץ המפתח שלכם (`.ppk`).
    6. לחצו **Open** והשאירו את החלון פתוח.

* **ב-macOS או Linux (באמצעות הטרמינל):**

    ```bash
    # החליפו את 'your-key.pem' ואת 'public-ip-address' בפרטים שלכם
    ssh -i "your-key.pem" -L 5901:localhost:5901 ec2-user@public-ip-address
    ```

    השאירו את הטרמינל הזה פתוח ברקע.

#### ב. חיבור עם VNC Client

1. התקינו VNC Client על מחשבכם (למשל, [TigerVNC Viewer](https://tigervnc.org/)).
2. פתחו את התוכנה ובשדה הכתובת הזינו: `localhost:5901`.
3. התחברו והזינו את הסיסמה שהגדרתם עם `vncpasswd`.

כעת אתם אמורים לראות את שולחן העבודה של GNOME. 🎉

-----

## שלב 6: הוספת משתמשים נוספים

ניתן להוסיף משתמשים נוספים ולאפשר לכל אחד מהם גישת SSH וסביבה גרפית משלו.

#### א. הוספת משתמש למערכת והגדרת SSH

1. **יצירת המשתמש בשרת:** התחברו ב-SSH לשרת והריצו פקודה זו (החליפו את `newuser` בשם המשתמש הרצוי):

    ```bash
    sudo adduser newuser
    ```

2. **קבלת מפתח ציבורי:** המשתמש החדש צריך ליצור זוג מפתחות SSH במחשב שלו ולשלוח לכם את **המפתח הציבורי** (בדרך כלל קובץ `id_rsa.pub`).
3. **שיוך המפתח הציבורי למשתמש:**

    ```bash
    # מעבר למשתמש החדש
    sudo su - newuser

    # צרו את תיקיית ה-ssh עבור המשתמש החדש
    mkdir .ssh

    # צרו את קובץ המפתחות המורשים
    touch .ssh/authorized_keys

    # הדביקו את המפתח הציבורי שקיבלתם לקובץ זה
    sudo vi .ssh/authorized_keys

    # הגדירו הרשאות מתאימות
    sudo chmod 700 .ssh
    sudo chmod 600 .ssh/authorized_keys
    ```

#### ב. הוספת המשתמש ל-VNC

1. **הוספת המשתמש לקובץ הגדרות ה-VNC:**
    ערכו את הקובץ והוסיפו שורה עבור המשתמש החדש עם מספר תצוגה ייחודי (למשל `:2`).

    ```bash
    sudo vi /etc/tigervnc/vncserver.users
    ```

      * הוסיפו שורה חדשה, למשל:

        ```bash
        :2=newuser
        ```

2. **הגדרת סיסמת VNC למשתמש החדש:**
    המשתמש החדש צריך להתחבר ב-SSH עם המשתמש שלו ולהריץ את `vncpasswd` בעצמו כדי להגדיר סיסמה אישית.

3. **הפעלת שירות ה-VNC עבור המשתמש החדש:**
    כמו שעשינו למשתמש `:1`, יש להפעיל ולאפשר את השירות עבור המשתמש החדש עם מספר התצוגה שלו (`:2`).

    ```bash
    sudo systemctl start vncserver@:2
    sudo systemctl enable vncserver@:2
    ```

4. **התחברות המשתמש החדש:**

      * המשתמש החדש יצטרך ליצור **מנהרת SSH** משלו, אך הפעם עם הפורט המתאים למספר התצוגה שלו. אם הוא משתמש בתצוגה `:2`, הפורט יהיה **5902**.

        ```bash
        # פקודת SSH למשתמש החדש
        ssh -i "path/to/his/key.pem" -L 5902:localhost:5902 newuser@public-ip-address
        ```

      * לאחר מכן, הוא יתחבר עם ה-VNC Client שלו לכתובת: `localhost:5902`.

---

Ref:
[Tutorial: Install the GNOME desktop environment on AL2023 - Amazon Linux 2023](https://docs.aws.amazon.com/linux/al2023/ug/installing-gnome-al2023.html)
[Tutorial: Configure TigerVNC server on AL2023 - Amazon Linux 2023](https://docs.aws.amazon.com/linux/al2023/ug/vnc-configuration-al2023.html)
[Manage system users on your Amazon EC2 Linux instance - Amazon Elastic Compute Cloud](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html)
