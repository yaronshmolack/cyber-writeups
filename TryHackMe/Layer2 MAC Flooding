# Layer2 MAC Flooding
**Platform:** TryHackMe  
**Lab URL:** https://tryhackme.com/room/layer2  

---

## Goal
- להבין כיצד מתקפת MAC Flooding יכולה לגרום ל-switch לשלוח תעבורה לכל הפורטים ב-VLAN.
- ללמוד לבצע passive sniffing של תעבורה בין hosts אחרים ברשת.
- לתעד את השינויים ב‑CAM table וההשפעה על הפקטות.

---

## Recon / Environment
1. **Network interfaces**  
   השתמשתי ב‑Nmap כדי לבדוק את כרטיסי הרשת במערכת:
   ```bash
   nmap --iflist 127.0.0.1

זה הציג את כל הממשקים, כולל eth1 (192.168.12.66/24) – הממשק אליו אני מחובר ברשת LAB.

חשיבות: tcpdump צריך לרוץ על eth1 כדי להאזין לתעבורה הרלוונטית; אם הייתי מריץ על ממשק אחר, לא הייתי מקבל חבילות.

Hosts discovery
בדקתי את /etc/hosts:

127.0.0.1       localhost
192.168.12.1    alice
192.168.12.2    bob
192.168.12.66   eve

יתרון: חסך סריקות אקטיביות ומיפוי ידני של כל מכונה ברשת.

Steps Taken

Passive sniffing לפני MAC flooding

tcpdump -i eth1

ראיתי תעבורה רק אלי.
שמרתי את הפקטות בקובץ pcap:

tcpdump -i eth1 -w /tmp/tcpdump.pcap

הורדתי ל‑host עם:

scp admin@10.48.165.58:/tmp/tcpdump.pcap .

ניתוח ב‑Wireshark אפשר לי לראות את גודל ה‑ICMP payload והפקטות.

MAC Flooding
הרצת macof כדי ליצור flood של כתובות MAC מזויפות אל ה‑CAM table של ה-switch.
אפקט: הטבלה התמלאה וה-switch לא הצליח למפות MACים חדשים → כל frame עם MAC לא ידוע נשלח לכל הפורטים ב‑VLAN.

Passive sniffing אחרי MAC flooding
לאחר ההתקפה, ראיתי תעבורה בין Alice ל‑Bob שהייתה חסויה לפני כן.
הסבר: ה-switch, מרוב בלבול ב‑CAM table, שלח את ה-frames לכל הפורטים ברשת.

Analysis

Before MAC Flooding: רק תעבורה שהייתה מיועדת לי נראתה.

After MAC Flooding: כל frame שנשלח ל-MAC לא מוכר הגיע אלי.

Mechanism: Overflow של CAM table גורם ל-switch לבצע unknown unicast flooding, מה שמאפשר Passive Sniffing.

Wireshark observation:

Frame sizes נותרו כפי שנשלחו (לדוגמה: 708 bytes / ICMP payload 666 bytes).

מה ששונה הוא ה‑forwarding behavior של ה-switch, לא גודל הפקטות.

Mitigation

Port Security / MAC limit:
הגבלת מספר MAC addresses לכל פורט → מונע overflow של CAM table.

Network Monitoring:

Port security violations

MAC flapping

Spike ב-unknown unicast flooding
נשלח ל-SIEM או monitored דרך Syslog/SNMP/NetFlow.

VLAN segmentation:
מגביל את ההשפעה של מתקפה ל‑VLAN ספציפי.

Lessons Learned

זיהוי ממשק רשת נכון הוא קריטי ל-sniffing.

CAM table מוגבלת; overflow גורם ל-switch להתנהג כמו hub.

Passive sniffing אחרי MAC flooding מראה כיצד מתקפות Layer2 יכולות לחשוף מידע ברשת.

חשוב להבין את ההבדל בין:

MAC Flooding (CAM table / switch behavior)

ARP Poisoning (ARP cache / host MITM)

