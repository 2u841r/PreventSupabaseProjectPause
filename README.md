# PreventSupabaseProjectPause

1. Visit your supabase project and get SUPABASE_URL, SUPABASE_KEY (SUPABASE_ANON_KEY)  
(for this, you have to click on <kbd> Connect </kbd> and click on App Framework then copy env.
2. Visit [Deno Deploy](https://dash.deno.com/account/overview) and click on New Playgroud.  
3. click on this icon

<img width="538" height="72" alt="image" src="https://github.com/user-attachments/assets/e543d0b2-cca5-48d3-bd0d-4d53393d3087" />

and add ENV values. 1. SUPABASE_URL1 2. SUPABASE_KEY1   

4. use this code - (but don't forget to adjust your env variable keys). this is for 2 project alive with this code.
if you have one project- then delete const supabase2 = createClient() and logTime(supabase2, "Database 2"),  

```ts
import { createClient } from "jsr:@supabase/supabase-js@2";

const supabase1 = createClient(
  Deno.env.get("SUPABASE_URL1")!,
  Deno.env.get("SUPABASE_KEY1")!,
);

const supabase2 = createClient(
  Deno.env.get("SUPABASE_URL2")!,
  Deno.env.get("SUPABASE_KEY2")!,
);

// Function to ensure table exists
async function ensureTableExists(supabase: any, tableName: string) {
  try {
    // Try to select from the table to check if it exists
    const { error } = await supabase.from(tableName).select("*").limit(1);
    
    if (error && error.code === "42P01") {
      // Table doesn't exist, create it
      console.log(`Creating table: ${tableName}`);
      const { error: createError } = await supabase.rpc("create_time_logs_table");
      
      if (createError) {
        console.error(`Error creating table ${tableName}:`, createError);
        return false;
      }
      console.log(`Table ${tableName} created successfully`);
    }
    return true;
  } catch (err) {
    console.error(`Error checking/creating table ${tableName}:`, err);
    return false;
  }
}

// Function to log time to database
async function logTime(supabase: any, dbName: string) {
  const tableName = "time_logs";
  
  // Ensure table exists
  const tableReady = await ensureTableExists(supabase, tableName);
  if (!tableReady) return;

  const currentTime = new Date().toISOString();
  
  try {
    const { data, error } = await supabase
      .from(tableName)
      .insert({ logged_at: currentTime })
      .select();

    if (error) {
      console.error(`Error inserting to ${dbName}:`, error);
    } else {
      console.log(`${dbName} - Time logged:`, currentTime);
    }
  } catch (err) {
    console.error(`Exception in ${dbName}:`, err);
  }
}

// Initialize tables on startup
console.log("Initializing databases...");
await Promise.all([
  logTime(supabase1, "Database 1"),
  logTime(supabase2, "Database 2"),
]);

Deno.cron("2MINCron", "*/2 * * * *", async () => {
  console.log("Cron triggered at:", new Date().toISOString());
  
  // Log time to both databases
  await Promise.all([
    logTime(supabase1, "Database 1"),
    logTime(supabase2, "Database 2"),
  ]);
});

console.log("Service started - logging time every 2 minutes");
```

5. create table and function on supabase Dashboard (Table Editor)  
```sql
   -- Create the time_logs table
CREATE TABLE IF NOT EXISTS time_logs (
  id SERIAL PRIMARY KEY,
  logged_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create a function to create the table (for the RPC call)
CREATE OR REPLACE FUNCTION create_time_logs_table()
RETURNS void
LANGUAGE sql
AS $$
  CREATE TABLE IF NOT EXISTS time_logs (
    id SERIAL PRIMARY KEY,
    logged_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
  );
$$;
```
6. Save & Deploy
