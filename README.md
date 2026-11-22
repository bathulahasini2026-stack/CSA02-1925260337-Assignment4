C CODE
#include <stdio.h>
#include <string.h>

typedef struct {
    int day, month, year;
} Date;

typedef struct {
    int systolic;
    int diastolic;
} BloodPressure;

typedef struct {
    char name[30];
    int age;
    char phone[11];       /* 10 digits plus null terminator */
    BloodPressure bp;     /* nested structure */
    Date visitDate;       /* nested structure */
} Patient;

void addPatient(Patient *p, const char *name, int age,
                const char *phone, BloodPressure bp, Date date)
{
    if (p == NULL || name == NULL || phone == NULL)
        return;

    snprintf(p->name, sizeof p->name, "%s", name);
    p->age = age;
    snprintf(p->phone, sizeof p->phone, "%s", phone);
    p->bp = bp;
    p->visitDate = date;
}

void printPatient(const Patient *p)
{
    printf("Name: %s\n", p->name);
    printf("Age: %d\n", p->age);
    printf("Phone: %s\n", p->phone);
    printf("Blood pressure: %d/%d\n",
           p->bp.systolic, p->bp.diastolic);
    printf("Visit date: %02d/%02d/%04d\n",
           p->visitDate.day, p->visitDate.month,
           p->visitDate.year);
}

int main(void)
{
    Patient patients[100];
    int count = 0;
    BloodPressure bp = {120, 80};
    Date visit = {22, 11, 2026};

    addPatient(&patients[count], "Anitha", 35,
               "9876543210", bp, visit);
    count++;

    printf("Patient register (%d record)\n\n", count);
    printPatient(&patients[0]);
    return 0;
}



PSEUDOCODE AND NOTES
PSEUDOCODE
1. Define Date with day, month and year.
2. Define BloodPressure with systolic and diastolic values.
3. Define Patient with name, age, phone, BloodPressure and Date.
4. Create an array of Patient structures and initialize count to zero.
5. To add a patient:
   a. Receive a pointer to a Patient and the patient details.
   b. Copy the name and phone into the structure fields.
   c. Assign age, blood pressure and visit date.
6. Increase count after a successful insertion.
7. Print the patient's details using the nested structure fields.

DESIGN DIAGNOSIS – DEFICIENCIES IN THE ORIGINAL PARALLEL ARRAYS
1. Parallel arrays can become misaligned if an element is inserted or deleted in only one array.
2. char phone[100][10] cannot store 10 digits plus the terminating '\0'; use char[11].
3. One float cannot represent both systolic and diastolic blood-pressure values.
4. A date stored as text is harder to validate and compare than day/month/year integer fields.
5. No capacity check can write beyond the 100 available records.
6. Global arrays and count make data changes less controlled and harder to test.
7. Sorting patients requires moving corresponding elements in every parallel array.

DESIGN DEFENCE
An array of structures keeps each patient's related fields together. A single structure can be passed to a function or moved during sorting, reducing the risk of mismatched fields. Nested Date and BloodPressure structures make the model clearer and reusable. The array capacity can be checked before insertion.

ALTERNATIVE / COMPLEXITY
Parallel arrays can be useful when processing one field at a time or when a columnar layout improves cache/vector processing. Sorting n records with a comparison sort is typically O(n log n); swaps in an array of structures move sizeof(Patient) bytes per swap, while parallel arrays must swap each corresponding field. Both designs store O(n) data. Structure padding may make sizeof(Patient) larger than the sum of field sizes; verify with sizeof.


EXECUTION OUTPUT
Patient register (1 record)

Name: Anitha
Age: 35
Phone: 9876543210
Blood pressure: 120/80
Visit date: 22/11/2026


OUTPUT SCREENSHOT
task01_output.png

==============================================================================
TASK 2 – NESTED STRUCTURES: COURIER PARCEL TRACKING
==============================================================================

C CODE
#include <stdio.h>
#include <string.h>

struct date { int day, month, year; };
struct address { char city[20]; int pincode; };
struct parcel {
    int id;
    float weight;
    struct address to;
    struct date booked;
};

void print_parcel(const struct parcel *p)
{
    printf("%d %.1f kg %s-%d %02d/%02d/%04d\n",
           p->id, p->weight, p->to.city, p->to.pincode,
           p->booked.day, p->booked.month, p->booked.year);
}

/* Selection sort in ascending order of weight. */
void sort_by_weight(struct parcel p[], int n)
{
    int i, j, minIndex;
    struct parcel temp;

    for (i = 0; i < n - 1; i++)
    {
        minIndex = i;
        for (j = i + 1; j < n; j++)
        {
            if (p[j].weight < p[minIndex].weight)
                minIndex = j;
        }

        if (minIndex != i)
        {
            temp = p[i];
            p[i] = p[minIndex];
            p[minIndex] = temp;
        }
    }
}

void list_by_receiver_city(const struct parcel p[], int n,
                           const char city[])
{
    int i;
    printf("Parcels for %s:\n", city);
    for (i = 0; i < n; i++)
    {
        if (strcmp(p[i].to.city, city) == 0)
            print_parcel(&p[i]);
    }
}

int main(void)
{
    struct parcel list[3] = {
        {501, 4.5f, {"Madurai", 625001}, {12, 10, 2026}},
        {502, 1.2f, {"Chennai", 600077}, {12, 10, 2026}},
        {503, 2.8f, {"Coimbatore", 641001}, {13, 10, 2026}}
    };

    sort_by_weight(list, 3);
    for (int i = 0; i < 3; i++)
        print_parcel(&list[i]);

    list_by_receiver_city(list, 3, "Chennai");

    printf("Third parcel receiver PIN (array/dot): %d\n",
           list[2].to.pincode);
    {
        struct parcel *ptr = &list[2];
        printf("Third parcel receiver PIN (pointer/arrow): %d\n",
               ptr->to.pincode);
    }
    return 0;
}



PSEUDOCODE AND NOTES
PSEUDOCODE – SORT PARCELS BY WEIGHT (SELECTION SORT)
1. For each position i from 0 to n-2:
   a. Assume i is the index of the lightest remaining parcel.
   b. Scan positions i+1 through n-1.
   c. If a lighter parcel is found, update minIndex.
   d. Swap the complete parcel structures at i and minIndex.
2. The array is now in ascending order of weight.

PSEUDOCODE – LIST PARCELS FOR A RECEIVER CITY
1. Read or receive the required city.
2. For each parcel in the array:
   a. Compare parcel.receiver.city with the required city.
   b. If equal, print that parcel.

STRUCTURE / ENTITY MODEL
- Date: day, month, year.
- Address: city, pincode.
- Parcel: id, weight, receiver address, booking date.
- Address and Date are nested inside Parcel.

ACCESS EXPRESSIONS
(i) Array and dot: list[2].to.pincode
(ii) Pointer and arrow: ptr->to.pincode

EXPECTED OUTPUT – REQUIRED TODO PROGRAM
502 1.2 kg Chennai-600077 12/10/2026
503 2.8 kg Coimbatore-641001 13/10/2026
501 4.5 kg Madurai-625001 12/10/2026

ALTERNATIVE / COMPLEXITY
A flat structure stores city and pincode and date fields directly in Parcel. Access becomes parcel.city and parcel.day rather than parcel.to.city and parcel.booked.day. A nested Address type is reusable by other structures. Selection sort takes O(n^2) comparisons and O(1) auxiliary space. Each swap copies sizeof(struct parcel) bytes. Use sizeof to compare nested and flat layouts because padding is implementation-dependent.


EXECUTION OUTPUT
502 1.2 kg Chennai-600077 12/10/2026
503 2.8 kg Coimbatore-641001 13/10/2026
501 4.5 kg Madurai-625001 12/10/2026
Parcels for Chennai:
502 1.2 kg Chennai-600077 12/10/2026
Third parcel receiver PIN (array/dot): 625001
Third parcel receiver PIN (pointer/arrow): 625001


OUTPUT SCREENSHOT
task02_output.png

==============================================================================
TASK 3 – UNIONS AND MEMORY LAYOUT: WEATHER-STATION PACKETS
==============================================================================

C CODE
#include <stdio.h>

enum kind { TEMPERATURE, HUMIDITY, STATUS };

struct reading {
    enum kind type;
    union {
        float celsius;
        int percent;
        char status[8];
    } value;
};

void show(const struct reading *r)
{
    switch (r->type)
    {
        case TEMPERATURE:
            printf("Temp %.1f C\n", r->value.celsius);
            break;
        case HUMIDITY:
            printf("Humidity %d %%\n", r->value.percent);
            break;
        case STATUS:
            printf("Status %s\n", r->value.status);
            break;
        default:
            printf("Unknown reading\n");
    }
}

int main(void)
{
    struct reading log[3] = {
        {TEMPERATURE, {.celsius = 31.5f}},
        {HUMIDITY, {.percent = 68}},
        {STATUS, {.status = "OK"}}
    };

    printf("sizeof union = %zu, sizeof struct reading = %zu\n",
           sizeof log[0].value, sizeof log[0]);

    for (int i = 0; i < 3; i++)
        show(&log[i]);

    return 0;
}



PSEUDOCODE AND NOTES
PSEUDOCODE – DISPLAY A TAGGED UNION
1. Receive a pointer to a Reading structure.
2. Examine the type tag.
3. If the tag is TEMPERATURE, print the celsius member.
4. If the tag is HUMIDITY, print the percent member.
5. If the tag is STATUS, print the status text.
6. For an unknown tag, print an error message.

MEMORY DESIGN
- A structure containing all three independent fields reserves space for all members.
- A tagged union stores a tag plus one shared memory area large enough for its largest member.
- The tag identifies which union member is currently valid.
- The union size is determined by its largest member plus any alignment requirements.
- On the assignment's typical 64-bit GCC setup, expected output is union = 8 bytes and struct reading = 12 bytes.

EXPECTED OUTPUT
sizeof union = 8, sizeof struct reading = 12
Temp 31.5 C
Humidity 68 %
Status OK

MISUSE EXPERIMENT
If a temperature is written and the humidity member is then read, the result is not a meaningful humidity value. The same memory is being interpreted as a different type. Always check the tag before reading a union member.

ALTERNATIVE / COMPLEXITY
Separate arrays by packet type avoid a tag check but need a way to associate records and may complicate ordering. A void-pointer design requires explicit type tracking and dynamic allocation management. A tagged union stores one packet in a compact, contiguous record. Reading and printing one packet is O(1); storage per packet is sizeof(struct reading).


EXECUTION OUTPUT
sizeof union = 8, sizeof struct reading = 12
Temp 31.5 C
Humidity 68 %
Status OK


OUTPUT SCREENSHOT
task03_output.png

==============================================================================
TASK 4 – STRUCTURE FORENSICS: BANK ACCOUNT
==============================================================================

C CODE
#include <stdio.h>

struct account {
    int number;
    float balance;
};

/* Demonstrates pass-by-value: a is a local copy. */
void addInterest_copy(struct account a, float rate)
{
    printf("Address inside pass-by-value function: %p\n", (void *)&a);
    a.balance = a.balance + a.balance * rate / 100.0f;
}

/* Correct version: modify the original using a structure pointer. */
void addInterest(struct account *a, float rate)
{
    if (a != NULL)
        a->balance = a->balance + a->balance * rate / 100.0f;
}

int main(void)
{
    struct account list[3] = {
        {101, 5000.0f}, {102, 12000.0f}, {103, 800.0f}
    };
    struct account *p = list;

    printf("Address in main: %p\n", (void *)&list[0]);
    addInterest_copy(list[0], 4.0f);
    printf("Balance after pass-by-value: %.2f\n", list[0].balance);

    for (int i = 0; i < 3; i++)
        addInterest(&list[i], 4.0f);

    printf("\nUpdated account balances:\n");
    for (int i = 0; i < 3; i++)
        printf("%d %.2f\n", p[i].number, p[i].balance);

    printf("First account using pointer: %d %.2f\n",
           p->number, p->balance);
    return 0;
}



PSEUDOCODE AND NOTES
PSEUDOCODE – REPAIR USING A STRUCTURE POINTER
1. Define Account with account number and balance.
2. Define addInterest to receive a pointer to Account and a rate.
3. If the pointer is not null:
   a. Access the balance using the arrow operator.
   b. Add balance * rate / 100 to the balance.
4. In main, create an array of accounts and a pointer to its first element.
5. Call addInterest with the address of each account.
6. Display all updated balances using the pointer.

DEFECTS IN THE ORIGINAL
1. Passing a structure by value creates a copy, so modifying the copy does not update the original array element.
2. p is a pointer, so p.number and p.balance are invalid. Use p->number and p->balance, or (*p).number and (*p).balance.

PASSING GUIDELINE
- Pass by value for small structures when a copy is wanted.
- Pass by pointer when a function must modify the original structure or avoid a large copy.
- Pass by const pointer when the function only reads a structure and should not modify it.

ALTERNATIVE / COMPLEXITY
A function can accept an Account by value, modify its local copy and return the modified Account; the caller assigns the returned value back. Both approaches take O(1) per account for this fixed-size structure. Passing by pointer passes an address; passing by value copies sizeof(struct account) bytes into the parameter and returns a structure copy. The pointer approach avoids those structure copies.


EXECUTION OUTPUT
Address in main: 0x7ffd9600b540
Address inside pass-by-value function: 0x7ffd9600b528
Balance after pass-by-value: 5000.00

Updated account balances:
101 5200.00
102 12480.00
103 832.00
First account using pointer: 101 5200.00


OUTPUT SCREENSHOT
task04_output.png

==============================================================================
TASK 5 – STORAGE CLASSES: TOKEN COUNTER
==============================================================================

C CODE
#include <stdio.h>

/* Defined in token_counter.c and shared through external linkage. */
extern int shared_token;

/* Issues the next token using a static local variable. */
int next_token(void)
{
    static int token = 1000;
    return ++token;
}

/* Automatic variable: reinitialized on every call. */
int auto_token(void)
{
    int token = 1000;
    return ++token;
}

/* Uses the global variable defined in token_counter.c. */
int extern_token(void)
{
    return ++shared_token;
}

int main(void)
{
    int s1 = next_token();
    int s2 = next_token();
    int s3 = next_token();

    int a1 = auto_token();
    int a2 = auto_token();
    int a3 = auto_token();

    int e1 = extern_token();
    int e2 = extern_token();
    int e3 = extern_token();

    printf("static: %d %d %d\n", s1, s2, s3);
    printf("auto  : %d %d %d\n", a1, a2, a3);
    printf("extern: %d %d %d\n", e1, e2, e3);

    printf("register loop: ");
    for (register int i = 1; i <= 3; i++)
        printf("%d ", i);
    printf("\n");

    return 0;
}


ADDITIONAL SOURCE FILE token_counter.c
int shared_token = 1000;


PSEUDOCODE AND NOTES
PSEUDOCODE – STATIC TOKEN COUNTER
1. Declare a local static token and initialize it to 1000 only once.
2. Increment token.
3. Return token.
4. Each later call continues from the previous value.

PSEUDOCODE – AUTOMATIC TOKEN COUNTER
1. On each function call, create a local automatic token initialized to 1000.
2. Increment token and return it.
3. The next call creates a fresh token initialized to 1000 again.

PSEUDOCODE – EXTERN / GLOBAL COUNTER
1. In token_counter.c, define shared_token and initialize it to 1000.
2. In task05_storage_classes.c, declare shared_token with extern.
3. Increment and return the shared variable.
4. Compile and link both source files together.

PSEUDOCODE – REGISTER LOOP VARIABLE
1. Declare the loop counter with register in the printing loop.
2. Print the counter while it is within the loop range.
3. The compiler may treat register as a hint; the program must not take its address.

EXPECTED OUTPUT
static: 1001 1002 1003
auto  : 1001 1001 1001
extern: 1001 1002 1003
register loop: 1 2 3

STORAGE CLASS COMPARISON
- auto: block scope, automatic lifetime, indeterminate initial value unless initialized, recreated on each call.
- static local: block scope, entire program lifetime, initialized once and retains its value.
- extern/global: external linkage for a shared definition, entire program lifetime, zero-initialized if not explicitly initialized.
- register: block scope and automatic lifetime; register is only a hint and its address cannot be taken with &.

TWO INDEPENDENT COUNTERS
Keep two separate static variables, two separate globals, or two caller-owned integers passed by pointer. Caller-owned counters are easiest to keep independent without hidden shared state.

ALTERNATIVE – POINTER COUNTER
next_token(int *counter): increment *counter and return it. The caller controls the counter's lifetime and can pass different counters for veg and non-veg. Time is O(1); extra function storage is O(1).


EXECUTION OUTPUT
static: 1001 1002 1003
auto  : 1001 1001 1001
extern: 1001 1002 1003
register loop: 1 2 3 


OUTPUT SCREENSHOT
task05_output.png

==============================================================================
TASK 6 – DESIGN CHALLENGE: CANTEEN INVENTORY AND BILLING
==============================================================================

C CODE
#include <stdio.h>
#include <string.h>

#define MAX_ITEMS 10
#define MAX_COMPONENTS 3

typedef struct {
    int day, month, year;
} Date;

typedef struct {
    int hour, minute;
} Time;

enum ItemType { PACKAGED, COOKED, COMBO };

typedef struct {
    int componentId[MAX_COMPONENTS];
    int quantity[MAX_COMPONENTS];
    int count;
} Combo;

typedef struct {
    int id;
    char name[30];
    enum ItemType type;
    float price;
    int stock;          /* used by packaged and cooked items */
    float revenue;
    union {
        Date expiry;
        Time prepared;
        Combo combo;
    } detail;
} Item;

int date_value(Date d)
{
    static const int daysBeforeMonth[12] =
        {0,31,59,90,120,151,181,212,243,273,304,334};
    int y = d.year - 1;
    int days = y * 365 + y / 4 - y / 100 + y / 400;
    days += daysBeforeMonth[d.month - 1] + d.day;
    if (d.month > 2 &&
        (d.year % 400 == 0 || (d.year % 4 == 0 && d.year % 100 != 0)))
        days++;
    return days;
}

Item *find_item(Item items[], int n, int id)
{
    for (int i = 0; i < n; i++)
        if (items[i].id == id)
            return &items[i];
    return NULL;
}

int record_sale(Item items[], int n, int id, int quantity)
{
    Item *item = find_item(items, n, id);
    if (item == NULL || quantity <= 0)
        return 0;

    if (item->type == COMBO)
    {
        Combo *c = &item->detail.combo;

        for (int i = 0; i < c->count; i++)
        {
            Item *part = find_item(items, n, c->componentId[i]);
            if (part == NULL || part->stock < c->quantity[i] * quantity)
                return 0;
        }

        for (int i = 0; i < c->count; i++)
        {
            Item *part = find_item(items, n, c->componentId[i]);
            part->stock -= c->quantity[i] * quantity;
        }
    }
    else
    {
        if (item->stock < quantity)
            return 0;
        item->stock -= quantity;
    }

    item->revenue += item->price * quantity;
    return 1;
}

void expiry_warning(const Item items[], int n, Date today)
{
    printf("Expiry warnings (within 2 days):\n");
    for (int i = 0; i < n; i++)
    {
        if (items[i].type == PACKAGED)
        {
            int remaining = date_value(items[i].detail.expiry) -
                            date_value(today);
            if (remaining >= 0 && remaining <= 2)
                printf("%s expires in %d day(s)\n",
                       items[i].name, remaining);
        }
    }
}

void print_revenue_summary(const Item items[], int n)
{
    int order[MAX_ITEMS], temp;

    for (int i = 0; i < n; i++)
        order[i] = i;

    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - i - 1; j++)
            if (items[order[j]].revenue <
                items[order[j + 1]].revenue)
            {
                temp = order[j];
                order[j] = order[j + 1];
                order[j + 1] = temp;
            }

    printf("\nDay bill summary (revenue descending):\n");
    for (int i = 0; i < n; i++)
        printf("%s : Rs. %.2f\n",
               items[order[i]].name, items[order[i]].revenue);
}

int main(void)
{
    Date today = {22, 11, 2026};
    Item items[3] = {
        {1, "Juice", PACKAGED, 30.0f, 5, 0.0f,
         {.expiry = {23, 11, 2026}}},
        {2, "Sandwich", COOKED, 50.0f, 2, 0.0f,
         {.prepared = {9, 30}}},
        {3, "ComboMeal", COMBO, 70.0f, 0, 0.0f,
         {.combo = {{1, 2, 0}, {1, 1, 0}, 2}}}
    };

    printf("Normal sale: ");
    if (record_sale(items, 3, 1, 2))
        printf("2 Juice sold\n");
    else
        printf("Sale rejected\n");

    printf("Excess-stock sale: ");
    if (record_sale(items, 3, 2, 99))
        printf("accepted\n");
    else
        printf("rejected - insufficient stock\n");

    printf("Combo sale: ");
    if (record_sale(items, 3, 3, 1))
        printf("1 ComboMeal sold\n");
    else
        printf("rejected - insufficient component stock\n");

    printf("Remaining Juice stock: %d\n", items[0].stock);
    printf("Remaining Sandwich stock: %d\n", items[1].stock);

    printf("\n");
    expiry_warning(items, 3, today);
    print_revenue_summary(items, 3);
    return 0;
}



PSEUDOCODE AND NOTES
PSEUDOCODE – RECORD A SALE
1. Find the item using its ID.
2. If the item does not exist or quantity is not positive, reject the sale.
3. If the item is a combo:
   a. Check stock for every component multiplied by the requested quantity.
   b. If any component has insufficient stock, reject without changing stock.
   c. Otherwise deduct all component quantities.
4. If the item is packaged or cooked:
   a. Check whether stock is sufficient.
   b. If not, reject the sale.
   c. Otherwise deduct the requested quantity.
5. Add price multiplied by quantity to that item's revenue.
6. Confirm the sale.

PSEUDOCODE – EXPIRY WARNING
1. Receive today's date.
2. Examine each inventory item.
3. For packaged items, calculate the number of days between today and expiry.
4. If expiry is from today through two days from today, print a warning.

PSEUDOCODE – BILL SUMMARY
1. Create an array of item indices.
2. Sort the indices by revenue in descending order.
3. Print each item and its revenue in that order.

DATA MODEL
- Date: day, month, year.
- Time: hour, minute.
- Combo: up to three component item IDs and quantities.
- Item: ID, name, type tag, price, stock, revenue and a tagged union.
- The union stores expiry Date, prepared Time or Combo data according to the item type.

TEST CASES INCLUDED
1. Normal sale: sell two Juice items.
2. Insufficient-stock sale: attempt to sell 99 Sandwiches; reject and do not change stock.
3. Combo sale: sell one ComboMeal and deduct its component stock.
4. Expiry warning: Juice expires within two days.
5. Revenue summary: sort item revenue from highest to lowest.

ALTERNATIVE – LINKED LIST
A linked list allows insertion and deletion at a known node in O(1), but finding an item by ID remains O(n). An array allows O(1) indexed access but searching by ID is O(n); insertion/deletion may require shifting elements. Each linked-list node uses extra memory for a next pointer. For a small fixed inventory, an array is simpler; for frequently changing inventory, a linked list may be useful.


EXECUTION OUTPUT
Normal sale: 2 Juice sold
Excess-stock sale: rejected - insufficient stock
Combo sale: 1 ComboMeal sold
Remaining Juice stock: 2
Remaining Sandwich stock: 1

Expiry warnings (within 2 days):
Juice expires in 1 day(s)

Day bill summary (revenue descending):
ComboMeal : Rs. 70.00
Juice : Rs. 60.00
Sandwich : Rs. 0.00


OUTPUT SCREENSHOT
task06_output.png
