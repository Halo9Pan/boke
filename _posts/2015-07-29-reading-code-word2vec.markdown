---
layout: post
title:  "Reading the word2vec code and refactory."
date:   2015-07-29 17:53:55
categories: Programming
---
主要读过[word2vec 中的数学原理详解（六）若干源码细节](http://blog.csdn.net/itplus/article/details/37999613)和[word2vec源码解析之word2vec.c](http://blog.csdn.net/lingerlanlan/article/details/38232755)，然后结合自己的理解。并对代码做了一定程度上的重构。


{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <stdbool.h>
#include <intrin.h>
#include <malloc.h>
#include <string.h>
#include <math.h>
#include <time.h>
#include <pthread.h>
#include <windows.h>

#define MAX_STRING 100
#define EXP_TABLE_SIZE 1000
#define MAX_EXP 6
#define MAX_SENTENCE_LENGTH 1000
#define MAX_CODE_LENGTH 40

const int vocabulary_hash_size = 30000000; // Maximum 30 * 0.7 = 21M words in the vocabulary

typedef float real;                    // Precision of float numbers

struct vocab_word
{
  int64_t frequency; //词频
  int32_t *point; //Huffman编码对应内节点的路径
  char *word; //词内容
  int8_t *code; //Huffman编码
  int8_t codelen; //Huffman编码长度
};

char train_file[MAX_STRING], output_file[MAX_STRING];
char save_vocab_file[MAX_STRING], read_vocab_file[MAX_STRING];
struct vocab_word *vocab;
int binary = 0, cbow = 1, debug_mode = 2, window = 5, min_count = 5, num_threads = 12, min_reduce = 1;
int *vocabulary_hash;
int64_t vocabulary_max_size = 1000, vocabulary_size = 0, layer1_size = 100; // vocabulary_size：词典大小，layer1_size：向量长度，隐含层的大小
int64_t train_words = 0, word_count_actual = 0, iter = 5, file_size = 0, classes = 0;
real alpha = 0.025, starting_alpha, sample = 1e-3;
real *syn0, *syn1, *syn1neg, *expTable;
// syn0：input -> hidden 的 weights，是一个1维数组，但是可以按照二维数组来理解。访问时实际上可以看成  syn0[i, j]，i为第i个单词，j为第j个隐含单元。大小：词典大小 * 隐含层大小
// syn1：hidden->output 的 weights
clock_t start;

int hs = 0, negative = 5;
const int table_size = 1e8;
int *table;

//每个单词的能量分布表，table在负样本抽样中用到
void
init_unigram_table ()
{
  int i, vocabulary_index;
  double train_words_pow = 0;
  double d1, power = 0.75;
  table = (int *) malloc (table_size * sizeof(int));
  for (i = 0; i < vocabulary_size; i++) //遍历词汇表，统计词的能量总值train_words_pow
    train_words_pow += pow (vocab[i].frequency, power);
  vocabulary_index = 0;
  d1 = pow (vocab[vocabulary_index].frequency, power) / train_words_pow;
  for (i = 0; i < table_size; i++)
    {
      table[i] = vocabulary_index;
      //table反映的是一个单词能量的分布，一个单词能量越大，所占用的table的位置越多
      if (i / (double) table_size > d1)
        {
          vocabulary_index++;
          d1 += pow (vocab[vocabulary_index].frequency, power) / train_words_pow;
        }
      if (vocabulary_index >= vocabulary_size)
        vocabulary_index = vocabulary_size - 1;
    }
}

// Reads a single word from a file, assuming space + tab + EOL to be word boundaries
//从文件中读取一个词，只有一个词
void
read_word (char *word, FILE *fin)
{
  int i = 0, ch;
  while (!feof (fin))
    {
      ch = fgetc (fin);
      if (ch == 13)
        continue;
      if ((ch == ' ') || (ch == '\t') || (ch == '\n'))
        {
          if (i > 0)
            {
              if (ch == '\n')
                ungetc (ch, fin);
              break;
            }
          if (ch == '\n')
            {
              strcpy (word, (char *) "</s>"); // 用"</s>"代替换行
              return;
            }
          else
            continue;
        }
      word[i] = ch;
      i++;
      if (i >= MAX_STRING - 1)
        i--;   // Truncate too long words
    }
  word[i] = 0;
}

// Returns hash value of a word
int
get_word_hash (char *word)
{
  uint64_t hash = 0;
  size_t i, size = strlen (word);
  for (i = 0; i < size; i++)
    {
      hash = hash * 257 + word[i];
    }
  hash = hash % vocabulary_hash_size;
  return hash;
}

// Returns position of a word in the vocabulary; if the word is not found, returns -1
// 通过hash表查找词，返回一个词在词汇表中的位置，如果不存在则返回-1
int
search_vocabulary (char *word)
{
  uint32_t hash = get_word_hash (word);
  while (true)
    {
      if (vocabulary_hash[hash] == -1)
        return -1;
      if (!strcmp (word, vocab[vocabulary_hash[hash]].word))
        return vocabulary_hash[hash];
      hash = (hash + 1) % vocabulary_hash_size;
    }
  return -1;
}

// Reads a word and returns its index in the vocabulary
// 从文件流中读取一个词，并返回这个词在词汇表中的位置
int
read_word_index (FILE *fin)
{
  char word[MAX_STRING];
  read_word (word, fin);
  if (feof (fin))
    return -1;
  return search_vocabulary (word);
}

// Adds a word to the vocabulary
// 将一个词添加到词表中
int
add_word_to_vocabulary (char *word)
{
  uint32_t hash, length = strlen (word) + 1;
  if (length > MAX_STRING)
    length = MAX_STRING;
  vocab[vocabulary_size].word = (char *) calloc (length, sizeof(char));
  strcpy (vocab[vocabulary_size].word, word);
  vocab[vocabulary_size].frequency = 0;
  vocabulary_size++;
  // Reallocate memory if needed
  if (vocabulary_size + 2 >= vocabulary_max_size)
    {
      vocabulary_max_size += 1000;
      vocab = (struct vocab_word *) realloc (vocab, vocabulary_max_size * sizeof(struct vocab_word));
    }
  hash = get_word_hash (word);
  while (vocabulary_hash[hash] != -1)
    hash = (hash + 1) % vocabulary_hash_size;
  vocabulary_hash[hash] = vocabulary_size - 1;
  return vocabulary_size - 1;
}

// Used later for sorting by word counts
int
vocabulary_compare (const void *a, const void *b)
{
  return ((struct vocab_word *) b)->frequency - ((struct vocab_word *) a)->frequency;
}

// Sorts the vocabulary by frequency using word counts
void
sort_vocabulary ()
{
  uint32_t i, size;
  uint32_t hash;
  // Sort the vocabulary and keep </s> at the first position
  // &vocab[1]用来跳过第一个</s>
  qsort (&vocab[1], vocabulary_size - 1, sizeof(struct vocab_word), vocabulary_compare);
  for (i = 0; i < vocabulary_hash_size; i++)
    vocabulary_hash[i] = -1; // 初始化hash表
  size = vocabulary_size;
  train_words = 0;
  for (i = 0; i < size; i++)
    {
      // Words occuring less than min_count times will be discarded from the vocab
      if ((vocab[i].frequency < min_count) && (i != 0))
        {
          vocabulary_size--;
          free (vocab[i].word); // 删除结构体内的内容
        }
      else
        {
          // Hash will be re-computed, as after the sorting it is not actual
          hash = get_word_hash (vocab[i].word);
          while (vocabulary_hash[hash] != -1)
            hash = (hash + 1) % vocabulary_hash_size;
          vocabulary_hash[hash] = i;
          train_words += vocab[i].frequency;
        }
    }
  vocab = (struct vocab_word *) realloc (vocab, (vocabulary_size + 1) * sizeof(struct vocab_word)); // 处理</s>
  // Allocate memory for the binary tree construction
  for (i = 0; i < vocabulary_size; i++)
    {
      vocab[i].code = (int8_t *) calloc (MAX_CODE_LENGTH, sizeof(char));
      vocab[i].point = (int32_t *) calloc (MAX_CODE_LENGTH, sizeof(int));
    }
}

// Reduces the vocabulary by removing infrequent tokens
// 再次移除词频过小的词，缩减词汇表
void
reduce_vocabulary ()
{
  int i, j = 0;
  uint32_t hash;
  for (i = 0; i < vocabulary_size; i++) // 双游标压缩
    {
      if (vocab[i].frequency > min_reduce)
        {
          vocab[j].frequency = vocab[i].frequency;
          vocab[j].word = vocab[i].word;
          j++;
        }
      else
        free (vocab[i].word); // 删除结构体内的内容
    }
  vocabulary_size = j;
  for (i = 0; i < vocabulary_hash_size; i++) // 重新计算hash值
    vocabulary_hash[i] = -1;
  for (i = 0; i < vocabulary_size; i++)
    {
      // Hash will be re-computed, as it is not actual
      hash = get_word_hash (vocab[i].word);
      while (vocabulary_hash[hash] != -1)
        hash = (hash + 1) % vocabulary_hash_size;
      vocabulary_hash[hash] = i;
    }
  fflush (stdout);
  min_reduce++; // 每次移除词频过小的词，自增一下
}

// Create binary Huffman tree using the word counts
// Frequent words will have short uniqe binary codes
void
create_binary_tree ()
{
  int64_t i, j, k, min1i, min2i, pos1, pos2, point[MAX_CODE_LENGTH];
  int8_t code[MAX_CODE_LENGTH];
  int64_t *count = (int64_t *) calloc (vocabulary_size * 2 + 1, sizeof(int64_t)); // calloc时置零
  int64_t *binary = (int64_t *) calloc (vocabulary_size * 2 + 1, sizeof(int64_t));
  int64_t *parent_node = (int64_t *) calloc (vocabulary_size * 2 + 1, sizeof(int64_t));
  for (i = 0; i < vocabulary_size; i++)
    count[i] = vocab[i].frequency; // 前半段词频
  for (i = vocabulary_size; i < vocabulary_size * 2; i++)
    count[i] = 1e15; // 后半段大值
  pos1 = vocabulary_size - 1;
  pos2 = vocabulary_size;
  // Following algorithm constructs the Huffman tree by adding one node at a time
  for (i = 0; i < vocabulary_size - 1; i++)
    {
      // First, find two smallest nodes 'min1, min2'
      if (pos1 >= 0) // 在count数组范围内查找
        {
          if (count[pos1] < count[pos2])
            {
              min1i = pos1;
              pos1--;
            }
          else
            {
              min1i = pos2;
              pos2++;
            }
        }
      else
        {
          min1i = pos2;
          pos2++;
        }
      if (pos1 >= 0) // 在count数组范围内查找
        {
          if (count[pos1] < count[pos2])
            {
              min2i = pos1;
              pos1--;
            }
          else
            {
              min2i = pos2;
              pos2++;
            }
        }
      else
        {
          min2i = pos2;
          pos2++;
        }
      count[vocabulary_size + i] = count[min1i] + count[min2i];
      parent_node[min1i] = vocabulary_size + i;
      parent_node[min2i] = vocabulary_size + i;
      binary[min2i] = 1; // 右子树为叶子节点
    }
  // Now assign binary code to each vocabulary word
  for (i = 0; i < vocabulary_size; i++)
    {
      j = i;
      k = 0;
      while (true)
        {
          code[k] = binary[j]; // 0或者1
          point[k] = j; // 路径
          k++;
          j = parent_node[j];
          if (j == vocabulary_size * 2 - 2)
            break;
        }
      vocab[i].codelen = k; // 树高
      vocab[i].point[0] = vocabulary_size - 2;
      for (j = 0; j < k; j++)
        {
          vocab[i].code[k - j - 1] = code[j];
          vocab[i].point[k - j] = point[j] - vocabulary_size;
        }
    }
  free (count);
  free (binary);
  free (parent_node);
}

void
learn_vocabulary_from_train_file ()
{
  char word[MAX_STRING];
  FILE *fin;
  int64_t i, index;
  for (i = 0; i < vocabulary_hash_size; i++)
    vocabulary_hash[i] = -1;
  fin = fopen (train_file, "rb");
  if (fin == NULL)
    {
      printf ("ERROR: training data file not found!\n");
      exit (EXIT_FAILURE);
    }
  vocabulary_size = 0;
  add_word_to_vocabulary ((char *) "</s>"); // 第一个词是</s>
  while (true)
    {
      read_word (word, fin);
      if (feof (fin))
        break;
      train_words++;
      if ((debug_mode > 1) && (train_words % 100000 == 0))
        {
          printf ("%lldK%c", train_words / 1000, 13);
          fflush (stdout);
        }
      index = search_vocabulary (word);
      if (index == -1) // word不在词汇表中
        {
          i = add_word_to_vocabulary (word); // 将word添加到词汇表
          vocab[i].frequency = 1; // 词频加1
        }
      else
        vocab[index].frequency++; // 词频自增
      if (vocabulary_size > vocabulary_hash_size * 0.7) // hash表占比超过70%时，减少词汇
        reduce_vocabulary ();
    }
  sort_vocabulary (); // 排序词汇表
  if (debug_mode > 0)
    {
      printf ("Vocabulary size: %lld\n", vocabulary_size);
      printf ("Words in train file: %lld\n", train_words);
    }
  file_size = ftell (fin);
  fclose (fin);
}

void
save_vocabulary ()
{
  int64_t i;
  FILE *fo = fopen (save_vocab_file, "wb"); // 二进制文件
  for (i = 0; i < vocabulary_size; i++)
    fprintf (fo, "%s %lld\n", vocab[i].word, vocab[i].frequency);
  fclose (fo);
}

void
read_vocabulary ()
{
  int64_t i = 0;
  char c;
  char word[MAX_STRING]; // word数组，用来保存读取到的词
  FILE *fin = fopen (read_vocab_file, "rb");
  if (fin == NULL)
    {
      printf ("Vocabulary file not found\n");
      exit (EXIT_FAILURE);
    }
  for (i = 0; i < vocabulary_hash_size; i++)
    vocabulary_hash[i] = -1;
  vocabulary_size = 0;
  while (true)
    {
      read_word (word, fin);
      if (feof (fin))
        break;
      i = add_word_to_vocabulary (word);
      fscanf (fin, "%lld%c", &vocab[i].frequency, &c);
    }
  sort_vocabulary ();
  if (debug_mode > 0)
    {
      printf ("Vocab size: %lld\n", vocabulary_size);
      printf ("Words in train file: %lld\n", train_words);
    }
  fin = fopen (train_file, "rb");
  if (fin == NULL)
    {
      printf ("ERROR: training data file not found!\n");
      exit (EXIT_FAILURE);
    }
  fseek (fin, 0, SEEK_END);
  file_size = ftell (fin);
  fclose (fin);
}

void
initialize_net ()
{
  int64_t a, b;
  uint64_t next_random = 1;
  size_t size = (size_t) vocabulary_size * layer1_size * sizeof(real);
  printf("size: %I64u\n", size);
  fflush(stdout);
  syn0 = _aligned_malloc (size, 128);
  if (syn0 == NULL)
    {
      printf ("Memory allocation failed with syn0\n");
      exit (EXIT_FAILURE);
    }
  if (hs)
    {
      syn1 = _aligned_malloc (size, 128);
      if (syn1 == NULL)
        {
          printf ("Memory allocation failed with syn1\n");
          exit (EXIT_FAILURE);
        }
      for (a = 0; a < vocabulary_size; a++)
        for (b = 0; b < layer1_size; b++)
          syn1[a * layer1_size + b] = 0;
    }
  if (negative > 0)
    {
      syn1neg = _aligned_malloc (size, 128);
      if (syn1neg == NULL)
        {
          printf ("Memory allocation failed with syn1neg\n");
          exit (EXIT_FAILURE);
        }
      for (a = 0; a < vocabulary_size; a++)
        for (b = 0; b < layer1_size; b++)
          syn1neg[a * layer1_size + b] = 0;
    }
  for (a = 0; a < vocabulary_size; a++)
    for (b = 0; b < layer1_size; b++)
      {
        next_random = next_random * (uint64_t) 25214903917 + 11;
        syn0[a * layer1_size + b] = (((next_random & 0xFFFF) / (real) 65536) - 0.5) / layer1_size; // syn0填充随机数
      }
  create_binary_tree();
}

void *
train_model_thread (void *id)
{
  int64_t a, b, d, cw, word_index, last_word_index, sentence_length = 0, sentence_position = 0;
  int64_t word_count = 0, last_word_count = 0, sentence[MAX_SENTENCE_LENGTH + 1];
  int64_t l1, l2, c, target, label, local_iter = iter;
  uint64_t next_random = (int64_t) id;
  real f, g;
  clock_t now;
  real *neu1 = (real *) calloc (layer1_size, sizeof(real)); // 隐含层神经的值， 大小： 隐含层大小
  real *neu1e = (real *) calloc (layer1_size, sizeof(real)); // 隐含层误差量， 大小： 隐含层大小
  FILE *fi = fopen (train_file, "rb");
  fseek (fi, file_size / (int64_t) num_threads * (int64_t) id, SEEK_SET);
  while (true)
    {
      if (word_count - last_word_count > 10000) // 每次处理10000词，此处可分布
        {
          word_count_actual += word_count - last_word_count;
          last_word_count = word_count;
          if ((debug_mode > 1))
            {
              now = clock ();
              printf ("%cAlpha: %f  Progress: %.2f%%  Words/thread/sec: %.2fk  ", 13, alpha,
                      word_count_actual / (real) (iter * train_words + 1) * 100,
                      word_count_actual / ((real) (now - start + 1) / (real) CLOCKS_PER_SEC * 1000));
              fflush (stdout);
            }
          alpha = starting_alpha * (1 - word_count_actual / (real) (iter * train_words + 1)); // 改变梯度步长，自适应学习率
          if (alpha < starting_alpha * 0.0001)
            alpha = starting_alpha * 0.0001;
        }
      if (sentence_length == 0)
        {
          while (true)
            {
              word_index = read_word_index (fi);
              if (feof (fi))
                break;
              if (word_index == -1)
                continue;
              word_count++;
              if (word_index == 0) // 换行 <s>时跳出
                break;
              // The subsampling randomly discards frequent words while keeping the ranking same
              if (sample > 0)
                {
                  real ran = (sqrt (vocab[word_index].frequency / (sample * train_words)) + 1) * (sample * train_words)
                      / vocab[word_index].frequency;
                  next_random = next_random * (uint64_t) 25214903917 + 11;
                  if (ran < (next_random & 0xFFFF) / (real) 65536)
                    continue; // 按一定概率舍去高频词
                }
              sentence[sentence_length] = word_index; // 组成句子，句子数组的值为word在词典中的index
              sentence_length++;
              if (sentence_length >= MAX_SENTENCE_LENGTH)
                break;
            }
          sentence_position = 0;
        }
      if (feof (fi) || (word_count > train_words / num_threads)) //
        {
          word_count_actual += word_count - last_word_count;
          local_iter--;
          if (local_iter == 0)
            break;
          word_count = 0;
          last_word_count = 0;
          sentence_length = 0;
          fseek (fi, file_size / (int64_t) num_threads * (int64_t) id, SEEK_SET);
          continue;
        }
      word_index = sentence[sentence_position];
      if (word_index == -1)
        continue;
      for (c = 0; c < layer1_size; c++)
        neu1[c] = 0;
      for (c = 0; c < layer1_size; c++)
        neu1e[c] = 0;
      next_random = next_random * (uint64_t) 25214903917 + 11;
      b = next_random % window;
      if (cbow)
        {  //train the cbow architecture
          // in -> hidden
          cw = 0;
          for (a = b; a < window * 2 + 1 - b; a++)
            if (a != window)
              {
                c = sentence_position - window + a;
                if (c < 0)
                  continue;
                if (c >= sentence_length)
                  continue;
                last_word_index = sentence[c];
                if (last_word_index == -1)
                  continue;
                for (c = 0; c < layer1_size; c++)
                  neu1[c] += syn0[c + last_word_index * layer1_size]; // 核心计算
                cw++;
              }
          if (cw)
            {
              for (c = 0; c < layer1_size; c++)
                neu1[c] /= cw;
              if (hs)
                for (d = 0; d < vocab[word_index].codelen; d++)
                  {
                    f = 0;
                    l2 = vocab[word_index].point[d] * layer1_size;
                    // Propagate hidden -> output
                    for (c = 0; c < layer1_size; c++)
                      f += neu1[c] * syn1[c + l2];
                    if (f <= -MAX_EXP)
                      continue;
                    else if (f >= MAX_EXP)
                      continue;
                    else
                      f = expTable[(int) ((f + MAX_EXP) * (EXP_TABLE_SIZE / MAX_EXP / 2))];
                    // 'g' is the gradient multiplied by the learning rate
                    g = (1 - vocab[word_index].code[d] - f) * alpha;
                    // Propagate errors output -> hidden
                    for (c = 0; c < layer1_size; c++)
                      neu1e[c] += g * syn1[c + l2];
                    // Learn weights hidden -> output
                    for (c = 0; c < layer1_size; c++)
                      syn1[c + l2] += g * neu1[c];
                  }
              // NEGATIVE SAMPLING
              if (negative > 0)
                for (d = 0; d < negative + 1; d++)
                  {
                    if (d == 0)
                      {
                        target = word_index;
                        label = 1;
                      }
                    else
                      {
                        next_random = next_random * (uint64_t) 25214903917 + 11;
                        target = table[(next_random >> 16) % table_size];
                        if (target == 0)
                          target = next_random % (vocabulary_size - 1) + 1;
                        if (target == word_index)
                          continue;
                        label = 0;
                      }
                    l2 = target * layer1_size;
                    f = 0;
                    for (c = 0; c < layer1_size; c++)
                      f += neu1[c] * syn1neg[c + l2];
                    if (f > MAX_EXP)
                      g = (label - 1) * alpha;
                    else if (f < -MAX_EXP)
                      g = (label - 0) * alpha;
                    else
                      g = (label - expTable[(int) ((f + MAX_EXP) * (EXP_TABLE_SIZE / MAX_EXP / 2))]) * alpha;
                    for (c = 0; c < layer1_size; c++)
                      neu1e[c] += g * syn1neg[c + l2];
                    for (c = 0; c < layer1_size; c++)
                      syn1neg[c + l2] += g * neu1[c];
                  }
              // hidden -> in
              for (a = b; a < window * 2 + 1 - b; a++)
                if (a != window)
                  {
                    c = sentence_position - window + a;
                    if (c < 0)
                      continue;
                    if (c >= sentence_length)
                      continue;
                    last_word_index = sentence[c];
                    if (last_word_index == -1)
                      continue;
                    for (c = 0; c < layer1_size; c++)
                      syn0[c + last_word_index * layer1_size] += neu1e[c];
                  }
            }
        }
      else
        {  //train skip-gram
          for (a = b; a < window * 2 + 1 - b; a++)
            if (a != window)
              {
                c = sentence_position - window + a;
                if (c < 0)
                  continue;
                if (c >= sentence_length)
                  continue;
                last_word_index = sentence[c];
                if (last_word_index == -1)
                  continue;
                l1 = last_word_index * layer1_size;
                for (c = 0; c < layer1_size; c++)
                  neu1e[c] = 0;
                // HIERARCHICAL SOFTMAX
                if (hs)
                  for (d = 0; d < vocab[word_index].codelen; d++)
                    {
                      f = 0;
                      l2 = vocab[word_index].point[d] * layer1_size;
                      // Propagate hidden -> output
                      for (c = 0; c < layer1_size; c++)
                        f += syn0[c + l1] * syn1[c + l2];
                      if (f <= -MAX_EXP)
                        continue;
                      else if (f >= MAX_EXP)
                        continue;
                      else
                        f = expTable[(int) ((f + MAX_EXP) * (EXP_TABLE_SIZE / MAX_EXP / 2))];
                      // 'g' is the gradient multiplied by the learning rate
                      g = (1 - vocab[word_index].code[d] - f) * alpha;
                      // Propagate errors output -> hidden
                      for (c = 0; c < layer1_size; c++)
                        neu1e[c] += g * syn1[c + l2];
                      // Learn weights hidden -> output
                      for (c = 0; c < layer1_size; c++)
                        syn1[c + l2] += g * syn0[c + l1];
                    }
                // NEGATIVE SAMPLING
                if (negative > 0)
                  for (d = 0; d < negative + 1; d++)
                    {
                      if (d == 0)
                        {
                          target = word_index;
                          label = 1;
                        }
                      else
                        {
                          next_random = next_random * (uint64_t) 25214903917 + 11;
                          target = table[(next_random >> 16) % table_size];
                          if (target == 0)
                            target = next_random % (vocabulary_size - 1) + 1;
                          if (target == word_index)
                            continue;
                          label = 0;
                        }
                      l2 = target * layer1_size;
                      f = 0;
                      for (c = 0; c < layer1_size; c++)
                        f += syn0[c + l1] * syn1neg[c + l2];
                      if (f > MAX_EXP)
                        g = (label - 1) * alpha;
                      else if (f < -MAX_EXP)
                        g = (label - 0) * alpha;
                      else
                        g = (label - expTable[(int) ((f + MAX_EXP) * (EXP_TABLE_SIZE / MAX_EXP / 2))]) * alpha;
                      for (c = 0; c < layer1_size; c++)
                        neu1e[c] += g * syn1neg[c + l2];
                      for (c = 0; c < layer1_size; c++)
                        syn1neg[c + l2] += g * syn0[c + l1];
                    }
                // Learn weights input -> hidden
                for (c = 0; c < layer1_size; c++)
                  syn0[c + l1] += neu1e[c];
              }
        }
      sentence_position++;
      if (sentence_position >= sentence_length)
        {
          sentence_length = 0;
          continue;
        }
    }
  fclose (fi);
  free (neu1);
  free (neu1e);
  pthread_exit (NULL);
}

void
TrainModel ()
{
  long a, b, c, d;
  FILE *fo;
  pthread_t *pt = (pthread_t *) malloc (num_threads * sizeof(pthread_t));
  printf ("Starting training using file %s\n", train_file);
  starting_alpha = alpha;
  if (read_vocab_file[0] != 0)
    read_vocabulary ();
  else
    learn_vocabulary_from_train_file ();
  if (save_vocab_file[0] != 0)
    save_vocabulary ();
  if (output_file[0] == 0)
    return;
  initialize_net ();
  if (negative > 0)
    init_unigram_table ();
  start = clock ();
  for (a = 0; a < num_threads; a++)
    pthread_create (&pt[a], NULL, train_model_thread, (void *) a);
  for (a = 0; a < num_threads; a++)
    pthread_join (pt[a], NULL);
  fo = fopen (output_file, "wb");
  if (classes == 0)
    {
      // Save the word vectors
      fprintf (fo, "%lld %lld\n", vocabulary_size, layer1_size);
      for (a = 0; a < vocabulary_size; a++)
        {
          fprintf (fo, "%s ", vocab[a].word);
          if (binary)
            for (b = 0; b < layer1_size; b++)
              fwrite (&syn0[a * layer1_size + b], sizeof(real), 1, fo);
          else
            for (b = 0; b < layer1_size; b++)
              fprintf (fo, "%lf ", syn0[a * layer1_size + b]);
          fprintf (fo, "\n");
        }
    }
  else
    {
      // Run K-means on the word vectors
      int clcn = classes, iter = 10, closeid;
      int *centcn = (int *) malloc (classes * sizeof(int));
      int *cl = (int *) calloc (vocabulary_size, sizeof(int));
      real closev, x;
      real *cent = (real *) calloc (classes * layer1_size, sizeof(real));
      for (a = 0; a < vocabulary_size; a++)
        cl[a] = a % clcn;
      for (a = 0; a < iter; a++)
        {
          for (b = 0; b < clcn * layer1_size; b++)
            cent[b] = 0;
          for (b = 0; b < clcn; b++)
            centcn[b] = 1;
          for (c = 0; c < vocabulary_size; c++)
            {
              for (d = 0; d < layer1_size; d++)
                cent[layer1_size * cl[c] + d] += syn0[c * layer1_size + d];
              centcn[cl[c]]++;
            }
          for (b = 0; b < clcn; b++)
            {
              closev = 0;
              for (c = 0; c < layer1_size; c++)
                {
                  cent[layer1_size * b + c] /= centcn[b];
                  closev += cent[layer1_size * b + c] * cent[layer1_size * b + c];
                }
              closev = sqrt (closev);
              for (c = 0; c < layer1_size; c++)
                cent[layer1_size * b + c] /= closev;
            }
          for (c = 0; c < vocabulary_size; c++)
            {
              closev = -10;
              closeid = 0;
              for (d = 0; d < clcn; d++)
                {
                  x = 0;
                  for (b = 0; b < layer1_size; b++)
                    x += cent[layer1_size * d + b] * syn0[c * layer1_size + b];
                  if (x > closev)
                    {
                      closev = x;
                      closeid = d;
                    }
                }
              cl[c] = closeid;
            }
        }
      // Save the K-means classes
      for (a = 0; a < vocabulary_size; a++)
        fprintf (fo, "%s %d\n", vocab[a].word, cl[a]);
      free (centcn);
      free (cent);
      free (cl);
    }
  fclose (fo);
}

int
ArgPos (char *str, int argc, char **argv)
{
  int a;
  for (a = 1; a < argc; a++)
    if (!strcmp (str, argv[a]))
      {
        if (a == argc - 1)
          {
            printf ("Argument missing for %s\n", str);
            exit (1);
          }
        return a;
      }
  return -1;
}

int
main (int argc, char **argv)
{
  int i;
  if (argc == 1)
    {
      printf ("WORD VECTOR estimation toolkit v 0.1c\n\n");
      printf ("Options:\n");
      printf ("Parameters for training:\n");
      printf ("\t-train <file>\n");
      printf ("\t\tUse text data from <file> to train the model\n");
      printf ("\t-output <file>\n");
      printf ("\t\tUse <file> to save the resulting word vectors / word clusters\n");
      printf ("\t-size <int>\n");
      printf ("\t\tSet size of word vectors; default is 100\n");
      printf ("\t-window <int>\n");
      printf ("\t\tSet max skip length between words; default is 5\n");
      printf ("\t-sample <float>\n");
      printf (
          "\t\tSet threshold for occurrence of words. Those that appear with higher frequency in the training data\n");
      printf ("\t\twill be randomly down-sampled; default is 1e-3, useful range is (0, 1e-5)\n");
      printf ("\t-hs <int>\n");
      printf ("\t\tUse Hierarchical Softmax; default is 0 (not used)\n");
      printf ("\t-negative <int>\n");
      printf ("\t\tNumber of negative examples; default is 5, common values are 3 - 10 (0 = not used)\n");
      printf ("\t-threads <int>\n");
      printf ("\t\tUse <int> threads (default 12)\n");
      printf ("\t-iter <int>\n");
      printf ("\t\tRun more training iterations (default 5)\n");
      printf ("\t-min-count <int>\n");
      printf ("\t\tThis will discard words that appear less than <int> times; default is 5\n");
      printf ("\t-alpha <float>\n");
      printf ("\t\tSet the starting learning rate; default is 0.025 for skip-gram and 0.05 for CBOW\n");
      printf ("\t-classes <int>\n");
      printf (
          "\t\tOutput word classes rather than word vectors; default number of classes is 0 (vectors are written)\n");
      printf ("\t-debug <int>\n");
      printf ("\t\tSet the debug mode (default = 2 = more info during training)\n");
      printf ("\t-binary <int>\n");
      printf ("\t\tSave the resulting vectors in binary moded; default is 0 (off)\n");
      printf ("\t-save-vocab <file>\n");
      printf ("\t\tThe vocabulary will be saved to <file>\n");
      printf ("\t-read-vocab <file>\n");
      printf ("\t\tThe vocabulary will be read from <file>, not constructed from the training data\n");
      printf ("\t-cbow <int>\n");
      printf ("\t\tUse the continuous bag of words model; default is 1 (use 0 for skip-gram model)\n");
      printf ("\nExamples:\n");
      printf (
          "./word2vec -train data.txt -output vec.txt -size 200 -window 5 -sample 1e-4 -negative 5 -hs 0 -binary 0 -cbow 1 -iter 3\n\n");
      return 0;
    }
  output_file[0] = 0;
  save_vocab_file[0] = 0;
  read_vocab_file[0] = 0;
  if ((i = ArgPos ((char *) "-size", argc, argv)) > 0)
    layer1_size = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-train", argc, argv)) > 0)
    strcpy (train_file, argv[i + 1]);
  if ((i = ArgPos ((char *) "-save-vocab", argc, argv)) > 0)
    strcpy (save_vocab_file, argv[i + 1]);
  if ((i = ArgPos ((char *) "-read-vocab", argc, argv)) > 0)
    strcpy (read_vocab_file, argv[i + 1]);
  if ((i = ArgPos ((char *) "-debug", argc, argv)) > 0)
    debug_mode = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-binary", argc, argv)) > 0)
    binary = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-cbow", argc, argv)) > 0)
    cbow = atoi (argv[i + 1]);
  if (cbow)
    alpha = 0.05;
  if ((i = ArgPos ((char *) "-alpha", argc, argv)) > 0)
    alpha = atof (argv[i + 1]);
  if ((i = ArgPos ((char *) "-output", argc, argv)) > 0)
    strcpy (output_file, argv[i + 1]);
  if ((i = ArgPos ((char *) "-window", argc, argv)) > 0)
    window = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-sample", argc, argv)) > 0)
    sample = atof (argv[i + 1]);
  if ((i = ArgPos ((char *) "-hs", argc, argv)) > 0)
    hs = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-negative", argc, argv)) > 0)
    negative = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-threads", argc, argv)) > 0)
    num_threads = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-iter", argc, argv)) > 0)
    iter = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-min-count", argc, argv)) > 0)
    min_count = atoi (argv[i + 1]);
  if ((i = ArgPos ((char *) "-classes", argc, argv)) > 0)
    classes = atoi (argv[i + 1]);
  vocab = (struct vocab_word *) calloc (vocabulary_max_size, sizeof(struct vocab_word));
  vocabulary_hash = (int *) calloc (vocabulary_hash_size, sizeof(int));
  expTable = (real *) malloc ((EXP_TABLE_SIZE + 1) * sizeof(real));
  for (i = 0; i < EXP_TABLE_SIZE; i++)
    {
      expTable[i] = exp ((i / (real) EXP_TABLE_SIZE * 2 - 1) * MAX_EXP); // Precompute the exp() table
      expTable[i] = expTable[i] / (expTable[i] + 1); // Precompute f(x) = x / (x + 1)
    }
  TrainModel ();
  return 0;
}

{% endhighlight %}

[info]:      http://halo9pan.info
[info-gh]:   http://github.halo9pan.info
[info-blog]: http://blog.halo9pan.info
