# superstore-api

## Technical Documentation

### Application Server

- Το runtime θα είναι γραμμένο σε Go, διότι διαθέτει minimal learning curve και επίσης διαθέτει τις καλύτερες native
  λύσεις σε επίπεδο runtime και όχι framework.
- Για τη δημιουργία των Rest APIs θα χρησιμοποιήσουμε ως web framework το Fiber. Frameworks όπως .NET και Spring Boot
  διαθέτουν πολύ μεγάλο learning curve όπως middlewares, security, pipelines etc. τα οποία εκτός από σύγχυση στον
  προγραμματιστή επιρεάζουν το performance - response time.
- Το .NET Native AOT θα ήταν μια αρκετά καλή λύση όμως δεν παρέχεται το reflection, το οποίο χρησιμοποιείται ίδη από
  μεγάλα packages όπως database drivers, EF Core etc.
- Τα Goroutines επίσης μας παρέχουν έναν από τους πιο efficient τρόπους για ασύγχρονο προγραμματισμό.

### Caching Layer

- Το application θα διαθέτει features όπως αναζήτηση προιόντων με βάση μια λίστα φίλτρων αλλά και εμφάνηση γραφιμάτων
  ανά προιών, κατηγορίες, προμηθευτές etc. Για αυτό ακριβώς θα χρειαστούμε ένα caching layer χωρίς license όπως για
  παράδειγμα το memcached. Αυτό που πρέπει να ελέγξουμε είναι το data synchronization μεταξύ των clusters το οποίο δε
  παρέχεται.

### Database Server

- Θα χρησιμοποιήσουμε ως βάση δεδομένων την Postgres διότι είναι η καλύτερη free licenced database αυτή τη στιγμή και
  υπάρχει αρκετά καλά γραμμένος driver για go runtime.

### Best Practices

- Όχι copy paste
- Χρησιμοποίησε τα καλύτερα design patterns όπως για παράδειγμα AuditManager (MBO) και TransferManager (Analog)
- Να χρησιμοποιούμε τα καλύτερα solutions εκεί που χρειάζονται (yield etc.).
- Εφόσον η εφαρμογή είναι κυρίως read data θα χρησιμοποιηθούν Triangle indexes.
- Για την αναζήτηση προιόντων θα χρησιμοποιηθεί ο αλγόριθμος Soundex (native support in latest Postgres versions).

### Payment Gateway

- Μπορούμε να ξεκινήσουμε με τη Stripe διότι αν και η πιο ακριβή διαθέτει το καλύτερο development experience και επίσης
  ήμαστε ίδη αρκετά familiar με τα documentation της, σε μεταγενέστερο χρόνο θα μπορούσαμε να δούμε την Adyen. Να ελέγξω
  τα benefit που θα διαθέτω αν επιλέξω ως Revolut τον deposit λογαριασμό.

### DevOps

- Integration Test (TestContainers, Wiremock)
- Benchmark - Stress test
- Kubernetes desktop

## Use cases

- Ένα scheduled background service θα πρέπει να εκτελείται κάθε μέρα ούτως ώστε να γίνει το scrapping στα site των
  διαθέσιμων merchant.
- Ο scraping μηχανισμός πρέπει εκτελείται σε 2 steps. Το superstore engine θα δέχεται ως input ένα merchant id το οποίο
  θα αντιστοιχεί σε ένα xml αρχείο. Το συγκεκριμένο αρχείο θα αναφέρονται όλα τα απαραίτητα actions που πρέπει να
  πραγματοποιηθούν ούτως ώστε να γίνουν retrieve τα προιόντα από το εκάστοτε site. Το output από το πρώτο step θα είναι
  ένα JSON αρχείο. Στο δεύτερο step θα πρέπει να γίνει επεξεργασία του αρχείου και αντιστοίχιση των merchant specific
  properties στο δικό μας domain. Για παράδειγμα υπάρχει περίπτωση τα οινοπνευματόδη ποτά να βρίσκονται στο site A σε
  multiple sections όπως (ουίσκι, βόκτες, κρασιά) ενώ στο site B να βρίσκονται σε ένα κοινό. Αυτό μπορεί να συμβεί διότι
  ο εκάστοτε merchant μπορεί έχει μια εξυδίκευση σε κάποια συγκεκριμένη κατηγορία προιόντος. Για αυτό το λόγω εμείς θα
  πρέπει να αντιστοιχίσουμε κάθε κατηγορία προιόν και κάθε προιόν με το δικό μας domain.
- Το output της παραπάνω διαδικασίας θα "περαστεί" στη Postgres database.
- Η βάση όπως αναφαίραμε θα χρειαστεί indexes (triangle etc.) διότι η εφαρμογή θα είναι κυρίως read και όχι
  write/update.
- Επίσης για να βοηθήσουμε την αναζήτηση των προιόντων από τον πελάτη θα πρέπει να υπάρχει ένα "έξυπνο" φίλτρο το οποίο
  θα κάνει fetch από τη βάση μέσω των indexed πεδίων. Για παράδειγμα ο χρήστης αν ξεκινήσει να πληκτρολογεί "λαχανικά"
  ή "vegetables" θα πρέπει να κάνουμε αναζήτηση στα soundex indexed πεδία (όνομα, κατηγορία etc.) και να του εμφανήσουμε
  ντομάτες αγγούρια κλπ. (μη γελάς μλκ).
- Services τα οποία διαθέτουν normal φίλτρα όπως αναζήτηση με βάση κατηγορία, θα μπορούσαν να υλοποιηθούν με ένα caching
  layer. Το expiration timestamp κάθε entry θα μπορούσε να είναι ακόμα και μετά από 24 ώρες διότι όπως αναφέραμε το
  scraping και κατεπέκταση το update της βάσης θα γίνεται 1 φορά τη μέρα.
- Η απόφαση του scraping schedule θα λαμβάνεται με βάση κάποια συγκεκριμένα κριτήρια (global και ανά merchant) όπως
  καθημερινό timestamp στο οποίο ο εκάστοτε merchant πραγματοποιεί ενημέρωση των δεδομένων στο site του. 