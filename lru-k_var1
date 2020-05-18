#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <time.h>

int contest = 0;                             // test variable

const int Correlated_Reference_Period = 0;
const int K = 3;                             // the last K references to popular database pages
const int keeping_time = 100;                // to keep track of the times
int t = 0;                                   // time
int Buff_Size = 20;

// the last reference to the page and history page
typedef struct page_t {
    int p;
    long long int last;
    long long int* hist;
} page_t;

// structure with a buffer and the lifetime of requests
typedef struct information_t {
    page_t* victim_buff;
    page_t* buff;
    int count_elem_in_buff;
    int count_elem_in_victim_buff;
    int victim_buff_size;
} information_t;


enum is_hit{
    MISS = 0,
    HIT = 1
};

//indicates whether this is a vict array or a buffer array
enum which_struct{
    ISBUFF = 0,
    ISVICT = 1
};



// function that initializes structure inf that keeps point on buffer,
// point on victim_buffer, amount of elements in both structures an size of Victim_buffer
void init_inf(information_t* inf);

// initialization page_t
void init_page(int refer, page_t* page);

// increases the size of the array
void victim_buff_increase(information_t* inf);

// decreases the size of the array
void victim_buff_decrease(information_t* inf);

// search for outdated links
int find_old_victim(information_t* inf, int* check_point);

// removing outdated links
void kick_old_victim(information_t* inf);

// link addition
void add_elem(int p, information_t* inf, int is_vic_buff);

// link search
int find_p(int p, page_t* arr, int count_of_elem);

void free_memory(information_t* inf);

//  LRU-k algorithm
int lru_k(int p, information_t* inf);

int test1()
{
    information_t inf;
    init_inf(&inf);
    int cashhit = 0;
    int i, j, size = 13000, p = 0;
    int list[size];

    srand(time(NULL));

    for (i=0; i<size; i++)
        list[i] = (rand() % 150)+1;

    for(i = 0; i<size; i++)
        cashhit += lru_k(list[i], &inf);


    printf("\ncashhit_rate = %f\n", ((float) cashhit)/((float) size));

    return 0;
}

int main() {
    information_t inf;
    int n = 0;
    int i;
//    scanf("%d", &Buff_Size);


    init_inf(&inf);

    test1();
/*    scanf("%d", &n);
    for (i = 0; i < n; i++) {
        int val;
        scanf("%d", &val);
        lru_k(val, &inf);
    }

    printf("%d", contest);  */
    free_memory(&inf);
}

// creates an array of structures page_t
void init_inf(information_t* inf) {
    inf->buff = (page_t*) calloc(Buff_Size, sizeof(page_t));
    inf->victim_buff = (page_t*) calloc(Buff_Size, sizeof(page_t));
    inf->count_elem_in_buff = 0;
    inf->count_elem_in_victim_buff = 0;
    inf->victim_buff_size = Buff_Size;
}

// storing information about the page
void init_page(int refer, page_t* page)
{
    int i;
    page->p = refer;
    page->hist = (long long int*) calloc(K, sizeof(long long int));
    page->last = t;
    page->hist[0] = t;

    for(i = 1; i < K; i++)
        page->hist[i] = 0;
}


void victim_buff_increase(information_t* inf)
{
    assert(inf->victim_buff != NULL);

    inf->victim_buff_size *= 2;

    inf->victim_buff = (page_t*) realloc(inf->victim_buff, sizeof(page_t) * inf->victim_buff_size);
    assert(inf->victim_buff != NULL);
}

void victim_buff_decrease(information_t* inf)
{
    assert(inf->victim_buff != NULL);

    inf->victim_buff_size /= 2;

    inf->victim_buff = (page_t*) realloc(inf->victim_buff, sizeof(page_t) * inf->victim_buff_size);
    assert(inf->victim_buff != NULL);
}

int find_old_victim(information_t* inf, int* checkpoint)
{
    for(;*checkpoint < inf->count_elem_in_victim_buff; (*checkpoint)++)
    {
        if((t - inf->victim_buff[*checkpoint].last) > keeping_time)
            return *checkpoint;
    }

    return -1;
}

void kick_old_victim(information_t* inf)
{
    int checkpoint = 0;
    assert(inf != NULL);

    for(checkpoint = find_old_victim(inf, &checkpoint); checkpoint != -1;)
    {
        inf->victim_buff[checkpoint] = inf->victim_buff[inf->count_elem_in_victim_buff];
        inf->count_elem_in_victim_buff -= 1;
        checkpoint--;
    }


    if(inf->count_elem_in_victim_buff < (inf->victim_buff_size / 3))
        victim_buff_decrease(inf);
}

void add_elem(int p, information_t* inf, int is_vic_buff)
{
    assert(inf->buff != NULL);
    assert(inf->victim_buff != NULL);


    if(is_vic_buff)
    {

        kick_old_victim(inf);

        if(inf->count_elem_in_victim_buff == inf->victim_buff_size)
            victim_buff_increase(inf);

        init_page(p, &inf->victim_buff[inf->count_elem_in_victim_buff]);
        inf->count_elem_in_victim_buff += 1;

    } else {
        init_page(p, &inf->buff[inf->count_elem_in_buff]);
        inf->count_elem_in_buff += 1;
    }
}

//searches for the p link
int find_p(int p, page_t* arr, int count_of_elem) {
    int i;
    for (i = 0; i < count_of_elem; i++) {
        if (arr[i].p == p) {
            return i;
        }
    }
    return -1;
}

void free_memory(information_t* inf) {
    int i;
    for (i = 0; i < inf->victim_buff_size; i++) {
        free(inf->victim_buff[i].hist);
    }
    for (i = 0; i < Buff_Size; i++) {
        free(inf->buff[i].hist);
    }
    free(inf->buff);
    free(inf->victim_buff);
}

//basic algorithm
int lru_k(int p, information_t* inf) {
    int pos_in_buff = find_p(p, inf->buff, inf->count_elem_in_buff);
    t++;
    if (pos_in_buff != -1) {
        contest++;
        if ((t - inf->buff[pos_in_buff].last) > Correlated_Reference_Period) {
            int i;
            long long int correl_period_of_refd_page = inf->buff[pos_in_buff].last - inf->buff[pos_in_buff].hist[0];
            for (i = 1; i < K; i++) {
                inf->buff[pos_in_buff].hist[i] = inf->buff[pos_in_buff].hist[i - 1] + correl_period_of_refd_page;
            }
            inf->buff[pos_in_buff].hist[0] = t;
            inf->buff[pos_in_buff].last = t;
        } else {
            inf->buff[pos_in_buff].last = t;
        }
//        printf("p = %d\n", p);
        return HIT;
    } else {
        if (inf->count_elem_in_buff < Buff_Size) {
            // write in inf->buff[inf->count_elem_in_buff] this page
            add_elem(p, inf, ISBUFF);
        } else {
            long long int min = t;
            int victim = -1;
            int q;
            int pos_in_victim_buff;

            for (q = 0; q < inf->count_elem_in_buff; q++) {

                //if there was no k last call for both links
                if(inf->buff[q].hist[K - 1] == min){
                    victim = (inf->buff[q].last - inf->buff[victim].last > 0) ? victim : q;
//                   printf("\n%d\n", inf->buff[victim].p);
                    continue;
                }

                if (((t - inf->buff[q].last) > Correlated_Reference_Period) && (inf->buff[q].hist[K - 1] < min)) {
                    victim = q;
                    min = inf->buff[q].hist[K - 1];
                }
            }

            assert(victim != -1);

            // adding victim to victim_buffer via the function
            add_elem(inf->buff[victim].p, inf, ISVICT);

            pos_in_victim_buff = find_p(p, inf->victim_buff, inf->count_elem_in_victim_buff);
            if (pos_in_victim_buff == -1) {
                //initialize and put in place victim
                init_page(p, &inf->buff[victim]);
            } else {
                int i;
                for (i = 1; i < K; i++) {
                    inf->victim_buff[pos_in_victim_buff].hist[i] = inf->victim_buff[pos_in_victim_buff].hist[i - 1];
                }
                inf->victim_buff[pos_in_victim_buff].hist[0] = t;
                inf->victim_buff[pos_in_victim_buff].last = t;

                inf->buff[victim] = inf->victim_buff[pos_in_victim_buff];

                inf->victim_buff[pos_in_victim_buff] = inf->victim_buff[inf->count_elem_in_victim_buff - 1];
                inf->count_elem_in_victim_buff -= 1;
            }

        }
        return MISS;
    }
}
