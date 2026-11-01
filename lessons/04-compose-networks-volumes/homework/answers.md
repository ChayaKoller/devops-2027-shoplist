# שיעור 4 — תשובות לשיעורי הבית (ד10)

## 1. איך Stats והדפדפן מגיעים ל־Backend ללא ports?
שירות Stats מגיע ל־Backend דרך הרשת הפנימית של Docker Compose באמצעות `http://backend:5000`. הדפדפן פונה ל־`localhost:8080`, ו־Nginx ב־Frontend מעביר את בקשות ה־API אל ה־Backend.

## 2. מה קורה לנתונים אחרי down ואחרי down -v?
אחרי `docker compose down` ואחריו `up`, המוצרים נשארים כי PostgreSQL שומר אותם ב־Named Volume בשם `postgres-data`. הפקודה `docker compose down -v` מוחקת גם את ה־Volume ולכן הנתונים נמחקים.

## 3. מה סוג ה־Volume של init.sql, ומה קורה אם משנים אותו?
החיבור `../sql/init.sql:/docker-entrypoint-initdb.d/init.sql` הוא **Bind Mount**, כי הוא מצביע על נתיב של קובץ בפרויקט במחשב. שינוי הקובץ לאחר שמסד הנתונים כבר אותחל לא יריץ אותו מחדש אוטומטית: סקריפטים בתיקיית האתחול מופעלים רק באתחול הראשון של תיקיית נתונים ריקה.

## 4. מה שם הרשת ומה שם ה־Volume?
שם ה־Volume בפועל הוא `docker_postgres-data`. שם הרשת של הפרויקט הוא `docker_default`, אך בעת הבדיקה היא לא הופיעה ב־`docker network ls`, כנראה משום שהופעלה `docker compose down` שמסירה את הרשת. Compose יוצר את השמות באמצעות שם הפרויקט (`docker`) בתוספת שם המשאב (`default` או `postgres-data`).

## 5. מה יש ב־Image הסופי של Multi-stage, ולמה אין CMD? למה Stats לא צריך Multi-stage?
ב־Image הסופי יש Nginx, קובצי האתר שנוצרו בתיקיית `dist` וקובץ `nginx.conf`; כלי הבנייה Node.js וקובצי המקור של שלב הבנייה אינם מועתקים אליו. אין צורך ב־`CMD` כי ל־Image הבסיסי של Nginx כבר יש פקודת הפעלה. בשירות Stats קוד Node.js עצמו נדרש בזמן הריצה, ולכן בתרגיל הזה אין צורך להפריד בין סביבת הבנייה לסביבת ההרצה.

## 6. גדלי שלושת ה־Images ולמה Multi קרוב ל־Single
- `single`: **92.8MB**
- `multi`: **92.7MB**
- `kitchen`: **244MB**

`multi` כמעט בגודל של `single` כי שניהם מבוססים בסופו של דבר על Nginx ומכילים רק את קובצי האתר וההגדרות. `kitchen` מכיל את סביבת Node.js ששימשה לבנייה, ולכן צפוי להיות גדול יותר.

## 7. איך Nginx בוחר location ומה עושה הלוכסן בסוף proxy_pass?
הנתיב `/api/stats` מתאים גם ל־`location /api/`, אבל Nginx בוחר את התאמת הקידומת הארוכה והספציפית יותר, `location /api/stats`, ולכן הבקשה מגיעה ל־Stats. ב־`proxy_pass http://backend:5000/` הלוכסן בסוף גורם להחלפת הקידומת `/api/` ב־`/`, ולכן בקשה ל־`/api/products` מועברת ל־Backend בתור `/products`.
