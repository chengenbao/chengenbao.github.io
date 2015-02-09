---
layout: post
title: Word Search
categories: Algorithm
---

Problem description
-------------------
Given a 2D board and a word, find if the word exists in the grid. 
The word can be constructed from letters of sequentially adjacent cell,
 where "adjacent" cells are those horizontally or vertically neighboring. 
The same letter cell may not be used more than once. 

For example, given board as:
{% highlight c++ %}
board = 
{
  {'A', 'B', 'C', 'E'},
  {'S', 'F', 'C', 'S'},
  {'A', 'D', 'E', 'E'}
}

word = "ABCCED", then return true;
word = "SEE", then return true;
word = "ABCB", then return false;
{% endhighlight %}

Solution
--------
{% highlight  c++ %}
class Solution 
{
  public:
    struct step
    {
      int x;
      int y;
      int direction;

      step(int x_pos = 0, int y_pos = 0, int d = 0) :
        x(x_pos), y(y_pos), direction(d){}

      step next()
      {
        step s;
        s.x = x;
        s.y = y;
        s.direction = 0;

        switch (direction)
        {
          case 0: // turn left
            s.y += 1;
            break;
          case 1: // down
            s.x += 1;
            break;
          case 2:
            s.y -= 1;
            break;
          case 3:
            s.x -= 1;
            break;
          default:
            break;
        }

        return s;
      }

      bool operator==(const step &rhs)
      {
        return x == rhs.x && y == rhs.y;
      }
    };

    bool exist(vector<vector<char> > &board, string word)
    {
      char board_letters[256];
      char word_letters[256];
      for (int i = 0; i < 256; ++i)
      {
        board_letters[i] = 0;
        word_letters[i] = 0;
      }

      for (int i = 0; i < board.size(); ++i)
      {
        for (int j = 0; j < board[i].size(); ++j)
        {
          ++board_letters[board[i][j]];
        }
      }

      for (int i = 0; i < word.length(); ++i)
      {
        ++word_letters[word[i]];
      }

      for (int i = 0; i < 256; ++i)
      {
        if (word_letters[i] > board_letters[i])
          return false;
      }

      step *track = new step[word.length()];

      for (int i = 0; i < board.size(); ++i)
      {
        for (int j = 0; j < board[i].size(); ++j)
        {
          if (word[0] != board[i][j])
            continue;

          int top = -1;
          step s(i, j, 0);
          // push to track
          track[++top] = s;
          while (top > -1)
          {
            s = track[top]; // get the last step
            if (top == word.length() - 1 && 
              word[top] == board[s.x][s.y]) // matched 
            {
              delete[]track;
              return true;
            }

            if (word[top] == board[s.x][s.y]) 
            {
              // go to the next step
              if (s.direction < 4)
              {
                step t = s.next();
                bool valid = false;
                if (t.x >= 0 && t.x < board.size() && 
                  t.y >= 0 && t.y < board[t.x].size())
                {
                  valid = true;
                }

                if (valid)
                {
                  bool find = false;
                  for (int k = top; k >= 0; --k)
                  {
                    if (track[k] == t)
                    {
                      find = true;
                      break;
                    }
                  }

                  if (!find)
                  {
                    track[++top] = t;
                  }
                  else
                  {
                    ++track[top].direction;
                  }
                }
                else
                {
                  ++track[top].direction;
                }
              }
              else // pop the top
              {
                --top;
                if (top > -1)
                  ++track[top].direction;
              }
            }
            else // do not match, try next direction
            {
              --top;
              if (top > -1)
                ++track[top].direction;
            }
          }
        }
      } // end for 
      delete[]track;

      return false;
    }
};
{% endhighlight %}

