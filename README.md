// src/SQLQueryAdvisor.java
import java.util.*;
import java.util.regex.*;

/**
 * SQLQueryAdvisor
 * - Heuristic parsing of simple SELECT queries and rule-based advice
 */
public class SQLQueryAdvisor {
    public static List<String> advise(String query) {
        String q = query.trim().toLowerCase();
        List<String> adv = new ArrayList<>();
        if (q.contains("select *")) adv.add("Avoid SELECT * — specify required columns to reduce IO.");
        Pattern p = Pattern.compile("from\\s+(\\w+)", Pattern.CASE_INSENSITIVE);
        Matcher m = p.matcher(query);
        if (m.find()) {
            adv.add("Detected table: " + m.group(1));
        }
        if (q.contains("where")) {
            adv.add("Query has WHERE — good. Check whether predicates are supported by indexes (equality predicates benefit most).");
            if (q.contains("like")) adv.add("LIKE may not use an index; consider trigram or full-text indexes if needed.");
        } else {
            adv.add("No WHERE clause — this will likely cause a full table scan.");
        }
        if (q.contains("order by")) adv.add("ORDER BY can be expensive — consider covering indexes or LIMIT pushdown.");
        return adv;
    }

    public static void main(String[] args) {
        String[] examples = {
            "SELECT * FROM users WHERE email = 'a@b.com';",
            "SELECT id, name FROM orders ORDER BY created_at DESC;",
            "SELECT count(*) FROM products;"
        };
        for (String e : examples) {
            System.out.println("Query: " + e);
            for (String a : advise(e)) System.out.println(" - " + a);
            System.out.println();
        }
    }
}
