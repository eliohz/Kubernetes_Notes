### Die Chef-Etage (Control Plane)

- **API-Server:** Die Anmeldung/Rezeption. Alles geht über ihn.
- **etcd:** Das goldene Notizbuch. Hier steht alles drin, was der Cluster wissen muss.
- **Controller Manager:** Die Aufseher. Sie prüfen: "Sollten hier nicht 3 Pods sein? Warum sind da nur 2? Mach neu!"
- **Scheduler:** Der Logistik-Planer. Er entscheidet: "Dieser Pod ist schwer, der kommt auf den starken Node 1."

### Die Werkstatt (Worker Node)

- **Kubelet:** Der Vorarbeiter auf dem Node. Er führt die Befehle der Chefs aus.
- **Kube-Proxy:** Der Postbote. Er sorgt dafür, dass die Netzwerkpakete beim richtigen Pod ankommen.
- **CRI (Runtime):** Der Motor (z. B. `containerd`), der die Container tatsächlich zum Laufen bringt.

---

## 2. Die Werkzeuge (Workload-Typen)

Je nachdem, was deine App tun soll, wählst du ein anderes Werkzeug:

| **Werkzeug**   | **Zweck**         | **Merkmal**                                      |
| -------------- | ----------------- | ------------------------------------------------ |
| **Deployment** | "Normaler" Dienst | Läuft dauerhaft, macht Updates ohne Pause.       |
| **ReplicaSet** | Der Bodyguard     | Achtet nur auf die Anzahl ("Immer 3!").          |
| **DaemonSet**  | Der Hausmeister   | Läuft auf **jedem** Node (z. B. für Logs).       |
| **Job**        | Der Kurier        | Erledigt eine Aufgabe und geht dann in Rente.    |
| **CronJob**    | Der Wecker        | Erledigt Aufgaben nach Zeitplan (z. B. Backups). |

---

## 3. Ordnung & Logik: Namespaces & Labels

- **Namespace:** Virtuelle Trennwände. `Dev`, `Test` und `Prod` wohnen im selben Haus, sehen sich aber nicht (Ordnung & Limits).
- **Labels & Selectors:** Das "Dating-System" von Kubernetes.
    - **Label:** Ein Sticker am Pod (`app=shop`).
    - **Selector:** Die Suchanfrage des Service ("Suche alle mit `app=shop`").
    - _Warum?_ So muss man keine IPs fest eintippen. Man sucht einfach nach Stickern.