# superstore-api

### Application Server:

- Το runtime θα είναι γραμμένο σε Go, διότι διαθέτει minimal learning curve και επίσης διαθέτει τις καλύτερες native
  λύσεις σε επίπεδο runtime και όχι framework.
- Για τη δημιουργία των Rest APIs θα χρησιμοποιήσουμε ως web framework το Fiber. Frameworks όπως .NET και Spring Boot
  διαθέτουν πολύ μεγάλο learning curve όπως middlewares, security, pipelines etc. τα οποία εκτός από σύγχυση στον
  προγραμματιστή επιρεάζουν το performance - response time.
- Το .NET Native AOT θα ήταν μια αρκετά καλή λύση όμως δεν παρέχεται το reflection, το οποίο χρησιμοποιείται ίδη από
  μεγάλα packages όπως database drivers, EF Core etc.
- Τα Goroutines επίσης μας παρέχουν έναν από τους πιο efficient τρόπους για ασύγχρονο προγραμματισμό.

### Caching Layer:

- Το application θα διαθέτει features όπως αναζήτηση προιόντων με βάση μια λίστα φίλτρων αλλά και εμφάνηση γραφιμάτων
  ανά προιών, κατηγορίες, προμηθευτές etc. Για αυτό ακριβώς τον θα χρειαστούμε ένα caching layer χωρίς license όπως για
  παράδειγμα το memcached. Αυτό που πρέπει να ελέγξουμε είναι το data synchronization μεταξύ των clusters.

### Database Server:

- Θα χρησιμοποιήσουμε ως βάση δεδομένων την Postgres διότι είναι η καλύτερη free licenced database αυτή τη στιγμή και
  υπάρχει αρκετά καλά γραμμένος driver για go runtime.

### Best Practices:

- Όχι copy paste
- Χρησιμοποίησε τα καλύτερα design patterns όπως για παράδειγμα ο AuditManager (MBO) και TransferManager (Analog)
- Να χρησιμοποιούμε τα καλύτερα solutions εκεί που χρειάζονται (yield etc.).

### Payment Gateway:

- Μπορούμε να ξεκινήσουμε με τη Stripe διότι αν και η πιο ακριβή διαθέτει το καλύτερο development experience και επίσης
  ήμαστε ίδη αρκετά familiar με τα documentation της, σε μεταγενέστερο χρόνο θα μπορούσαμε να δούμε την Adyen. Να ελέγξω
  τα benefit που θα διαθέτω αν επιλέξω ως Revolut τον deposit λογαριασμό. 