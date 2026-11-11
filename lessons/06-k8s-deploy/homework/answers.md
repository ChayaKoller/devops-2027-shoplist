# שיעור 6 - תשובות

## 1
א: את השם `docker-backend` נתן Compose באופן אוטומטי, לפי שם התיקייה שממנה הרצנו אותו (`docker`). השם הזה לא קבוע: בתיקייה אחרת הוא היה יוצא שונה. בקבצי ה-YAML צריך לכתוב שם קבוע וברור, ולכן בנינו את ה-images בעצמנו עם שם ותג שבחרנו: `shoplist-backend:local`.

ב: בלי `imagePullPolicy: Never`, Kubernetes היה מנסה להוריד את ה-image מהאינטרנט (מ-Docker Hub). ה-image `shoplist-backend:local` קיים רק במחשב שלי, ולכן ההורדה הייתה נכשלת וה-pod היה נתקע בשגיאה `ErrImagePull`.

## 2
א: קודם הצגתי את הערך המקודד עם `kubectl get secret postgres-secret -n shoplist -o yaml`, ואז פענחתי אותו ב-PowerShell:
`[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("c2hvcHBhc3M="))`
התוצאה: `shoppass`.

ב: Secret לא מוצפן. base64 הוא קידוד, לא הצפנה: כל מי שקוראת את ה-Secret מפענחת אותו בפקודה אחת. ההגנה האמיתית היא הרשאות (RBAC), ובסביבות אמיתיות גם הצפנה של etcd או כלי חיצוני לניהול סודות.

## 3
א: הקובץ `postgres-pvc.yml`. PVC הוא הבקשה לאחסון, PV הוא האחסון עצמו, ו-StorageClass מייצר PV לפי בקשה. כשה-pod נמחק, ה-PVC וה-PV נשארים, וה-pod החדש מתחבר אליהם, ולכן המוצרים נשארו.

ב: הנתונים נמחקים רק כשמוחקים את ה-PVC: `kubectl delete pvc postgres-pvc -n shoplist`. (גם `kubectl delete -f kubernetes/` היה מוחק אותם, כי קובץ ה-PVC נמצא בתיקייה.)

## 4
א: ב-Kubernetes אין סדר עלייה. אם ה-backend עולה לפני postgres, הוא נכשל בהתחלה, ו-Kubernetes ממשיך לנסות שוב עד שהכל מוכן.

ב: ה-`readinessProbe`. הוא בודק אם ה-pod מוכן לקבל בקשות (אצל ה-backend: `/health` בפורט 5000). כל עוד הבדיקה לא עוברת, ה-pod מסומן `0/1`, וה-Service לא שולח אליו בקשות.

## 5
אחרי ששיניתי את ה-selector ב-`backend-service.yml` ל-`app: backnd`, הנורית של backend בדפדפן הפכה לאדומה והאפליקציה הפסיקה לקבל מוצרים. ה-pods של backend **לא** נפלו, הם המשיכו לרוץ ב-`1/1`. אבל `kubectl get endpoints backend` הראה שאין כתובות (`<none>`), כי ה-Service מחפש pods עם המדבקה `backnd`, ואין אף pod כזה. ה-Service נוצר בלי שגיאה, אבל אין מאחוריו אף pod. אחרי שתיקנתי ל-`app: backend`, הכתובות חזרו והנורית חזרה לירוק.

## 6
משתני סביבה נקראים רק כשה-pod עולה. שינוי ב-ConfigMap לא משנה pods שכבר רצים, ולכן ה-backend ממשיך לראות את הערך הישן. הפתרון הוא להרים את ה-pods מחדש:
`kubectl rollout restart deployment/backend -n shoplist`

## 7
א: 16 פעמים: בשורה `namespace: shoplist` בכל הקבצים, ובשמות ה-images (למשל `shoplist-backend:local`).

ב: צריך לשנות 12 קבצים, כל קובץ שיש בו `namespace: shoplist` (ואחרי ד7 גם `app-config.yml`, כלומר 13). את שמות ה-images לא צריך לשנות, כי זה אותו קוד. זו בעיה: ערך שחוזר בהרבה קבצים קל לשכוח לשנות באחד מהם. את זה פותרים בשיעור 7 עם Helm.
