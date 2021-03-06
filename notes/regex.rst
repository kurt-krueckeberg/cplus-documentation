.. include:: <isopub.txt>

C++ Regular Expressions Functions
=================================

regex_match
-----------

``regex_match`` returns true and only if the entire input sequence **[start, end)** is matched. For example, the output of

.. code-block:: cpp

    void test_regex_match(const std::string& subject, const std::string& re_str)
    {
      regex re{re_str};

      cout << "Does subject of: '" << subject << "'. Match the regex of '" << re_str << "'" << endl;
    
      string msg { regex_match(subject, re) ? "yes" : "no" };
    
      cout << "Answer: " << msg << endl;
    }

    regex re{string{R"(\d\d/\d\d/\d\d\d\d)"}}; // four digit month/day/year date

    test_regex_match(string{"5/31/2000"}, R"(\d\d/\d\d/\d\d\d\d)"); // four digit month/day/year date

    test_regex_match(string{"05/31/2000"}, R"(\d\d/\d\d/\d\d\d\d)"); // four digit month/day/year date
    
is

::

    Does subject of: '5/31/2000'. Match the regex of '\d\d/\d\d/\d\d\d\d'
    Answer: no
    Does subject of: '05/31/2000'. Match the regex of '\d\d/\d\d/\d\d\d\d'
    Answer: yes 

regex_search and smatch
-----------------------

While ``regex_match`` returns true only if the regular expression matches the entire input sequence, ``regex_search`` will succeed even if only a sub-sequence matches the regular expression. And ``regex_search`` will return both the matched regex
and any submatches, any captures, within the regex. Given code such as

.. code-block:: cpp

    void test_regex_search()  
    {
      // [:w:] stands for "word character". It matches the ASCII characters [A-Za-z0-9_].
      // This regex matches a 
      regex re{R"(([[:w:]\.]+)@([[:w:]]+)\.com)"};
    
      smatch m;
    
      string s{R"(<prefix>kurt.krueckeberg@gmail.com<suffix>joe.smith@aol.com|jane.doe@comcast.net)"};
    
      if (regex_search(s, m, re)) { 

          for (auto i = 0; i < m.size(); ++i) {
        
              cout << "The " << i << "th submatch using m[i].str()     is: " <<  m[i].str() << endl;
              cout << "The " << i << "th submatch using m.str(i)       is: " <<  m.str(i) << endl;
              cout << "The " << i << "th submatch using *(begin() + n) is: " <<  *(m.begin() + i) << endl;
          }
          
          cout << "m.prefix().str() = " << m.prefix().str() << endl;
          
          cout << "m.suffix().str() = " << m.suffix().str() << endl;
      } 
    }

the first "submatch", ``m[0].str()`` and ``m.str(0)``, is always the match of the entire regex , while is the first submatch is returned in ``m[1].str()``. Submatches can be returned various ways that are all equivalent to one another

.. code-block:: cpp

    m[i].str() 
    m.str(i) 
    *(m.begin() + i)

In addition to returning submatches, ``regex_search`` returns everything before the matched expression in ``m.prefix().str()``, and everything after the matched expression in ``m.suffix().str()``.  Therefore ``m.prefix().str()`` contains **<prefix>**
and ``m.suffix().cstr()`` contains **<suffix>**.

In the code above ``regex_search()`` only found the first email address.

You can use ``regex_search()`` in a loop, but you must supply ``m.suffix().str()`` as the new input each time through the loop:
 
.. code-block:: cpp

    void test_regex_search2()  // See https://www.youtube.com/watch?v=nkjUpUu3dFk
    {
      regex re{R"(([[:w:]\.]+)@([[:w:]]+)\.com)"};
    
      smatch m;
    
      string s{R"(<prefix>kurt.krueckeberg@gmail.com<suffix>joe.smith@aol.com|jane.doe@comcast.net)"};

      for(;(regex_search(s, m, re) ); s = m.suffix().str())  { 
    
          for (auto i = 0; i < m.size(); ++i) {
        
              cout << "The " << i << "th submatch using m[i].str()     is: " <<  m[i].str() << endl;
              cout << "The " << i << "th submatch using m.str(i)       is: " <<  m.str(i) << endl;
              cout << "The " << i << "th submatch using *(begin() + n) is: " <<  *(m.begin() + i) << endl;
          }
          
          cout << "m.prefix().str() = " << m.prefix().str() << endl;
          
          cout << "m.suffix().str() = " << m.suffix().str() << endl;
        }
    }

The output is

.. raw:: html

    <pre>
    The 0th submatch using m[i].str()     is: kurt.krueckeberg@gmail.com
    The 0th submatch using m.str(i)       is: kurt.krueckeberg@gmail.com
    The 0th submatch using *(begin() + n) is: kurt.krueckeberg@gmail.com
    The 1th submatch using m[i].str()     is: kurt.krueckeberg
    The 1th submatch using m.str(i)       is: kurt.krueckeberg
    The 1th submatch using *(begin() + n) is: kurt.krueckeberg
    The 2th submatch using m[i].str()     is: gmail
    The 2th submatch using m.str(i)       is: gmail
    The 2th submatch using *(begin() + n) is: gmail
    m.prefix().str() = <prefix>
    m.suffix().str() = <suffix>joe.smith@aol.com|jane.doe@comcast.net
    The 0th submatch using m[i].str()     is: joe.smith@aol.com
    The 0th submatch using m.str(i)       is: joe.smith@aol.com
    The 0th submatch using *(begin() + n) is: joe.smith@aol.com
    The 1th submatch using m[i].str()     is: joe.smith
    The 1th submatch using m.str(i)       is: joe.smith
    The 1th submatch using *(begin() + n) is: joe.smith
    The 2th submatch using m[i].str()     is: aol
    The 2th submatch using m.str(i)       is: aol
    The 2th submatch using *(begin() + n) is: aol
    m.prefix().str() = <suffix>
    m.suffix().str() = |jane.doe@comcast.net
    </pre>
    
regex iterators
---------------

regex_iterator
^^^^^^^^^^^^^^

If we change ``s`` in the previous code above to be 

.. code-block:: cpp

    string s{R"(<prefix1>kurt.krueckeberg@gmail.com<suffix1>  <prefix2>jane.doe@yahoo.com<suffix2>)"}; 

    smatch m;

    auto found = regex_search(s, m, re);

``regex_search()`` will only find the first occurance in the string. To do repeated, iterative searching involving ``regex_search()``, we must use the while loop idiom shown in ``test_regex_search2()``:

 .. code-block:: cpp
 
    while(regex_search(s, m, re) ) {
    
      for (auto& x : m) {

         // handle submatches here...
      }  
      s = m.suffix().str();
    }

``regex_iterator`` is a more natural alternative:

.. code-block:: cpp

    void test_regex_iterator()
    {
      string s{R"(<prefix1>kurt.krueckeberg@gmail.com<suffix1>  <prefix2>kathafalk@yahoo.com<suffix2>)"};
      
      regex re{R"(([[:w:]\.]+)@([[:w:]]+)\.com)"};
      
      sregex_iterator re_iter(s.begin(), s.end(), re);
    
      sregex_iterator re_end;
    
      for (; re_iter != re_end; ++re_iter) {
    
          cout << "The 0th match using re_iter->str(0)\t is: " <<  re_iter->str(0) << " or the entire matched expression." << endl;
          cout << "The 1th submatch using re_iter->str(1)  is: " <<  re_iter->str(1) << endl;
          cout << "The 2th submatch using re_iter->str(2)  is: " <<  re_iter->str(2) << endl;
    
          cout << "The 2th submatch using re_iter->prefix()  is: " <<  re_iter->prefix() << endl;
          cout << "The 2th submatch using re_iter->suffix()  is: " <<  re_iter->suffix() << endl;
      }
    }
    
whose output is:

.. raw:: html

    <pre>
    The 0th match using re_iter->str(0)      is: kurt.krueckeberg@gmail.com or the entire matched expression.
    The 1th submatch using re_iter->str(1)  is: kurt.krueckeberg
    The 2th submatch using re_iter->str(2)  is: gmail
    The 2th submatch using re_iter->prefix()  is: <prefix1>
    The 2th submatch using re_iter->suffix()  is: <suffix1>  <prefix2>kathafalk@yahoo.com<suffix2>
    The 0th match using re_iter->str(0)      is: kathafalk@yahoo.com or the entire matched expression.
    The 1th submatch using re_iter->str(1)  is: kathafalk
    The 2th submatch using re_iter->str(2)  is: yahoo
    The 2th submatch using re_iter->prefix()  is: <suffix1>  <prefix2>
    The 2th submatch using re_iter->suffix()  is: <suffix2>
    </pre>
    
regex_token_iterator
^^^^^^^^^^^^^^^^^^^^

The other type of regex iterator is ``regex_token_iterator``. ``regex_iterator`` points to matched results. While ``regex_token_iterator`` points to submatches. Its
``str()`` method, unlike regex_iterator, cannot take a index. Thus ``regex_iter->str(i)`` works for regex_iterator objects, but for regex_token_iterator, we are limited
to ``regex_iter->str()``. For exmaple

.. code-block:: cpp

    void test_regex_token_iterator()
    {
      string s{R"(<prefix1>kurt.krueckeberg@gmail.com<suffix1>  <prefix2>kathafalk@yahoo.com<suffix2>)"};
      
      regex re{R"(([[:w:]\.]+)@([[:w:]]+)\.com)"};
      
      sregex_token_iterator re_iter(s.begin(), s.end(), re);
    
      sregex_token_iterator re_end;
    
      for (; re_iter != re_end; ++re_iter) {
    
          cout << "re_iter->str() is: " <<  re_iter->str() << endl;
      }
    }

whose output is:

[ADD OUTPUT HERE]

In addition regex_token_iterator can take a 4th parameter to control the way the matched result is returned. It can be either an int, indicating the submatch to return,
or an initializer list of ints, indicating the set of submatches to return. For example:

.. code-block:: cpp

    void run_regex_token_iterator()
    {
      string s {"this subject has a submarine as a subsequence"};
    
      cout << "\nSubject: " << s << "\n";
    
      try {
    
          cout << "\nregex: " << R"(\b(sub)([^ ]*))" << "\n" << endl;
          
          regex re {R"(\b(sub)([^ ]*))"};   
          
          sregex_token_iterator rend;
          
          string subject ("This subject has a submarine as a subsequence");
          
          show_regex( sregex_token_iterator{subject.begin(), subject.end(), re}, rend, "entire matchesubject");
          
          show_regex( sregex_token_iterator{subject.begin(), subject.end(), re, 0}, rend, "0 returns entire matched result ");
          
          show_regex( sregex_token_iterator{subject.begin(), subject.end(), re, 2}, rend, "2 returns 2nd submatch");
          
          show_regex( sregex_token_iterator{subject.begin(), subject.end(), re, {1, 2}}, rend, "{1, 2} returns 1st and 2nd submatches");
          
          show_regex( sregex_token_iterator{subject.begin(), subject.end(), re, -1}, rend, "-1 returns nonmatched text");
          
          show_regex( sregex_token_iterator{subject.begin(), subject.end(), re, {-1, 0}}, rend, "{-1, 0} returns nonmatched text followed by entire match");
    
          show_regex( sregex_token_iterator{subject.begin(), subject.end(), re, {-1, 0, 1}}, rend, "{-1, 0, 1} matchesubject");
      
      } catch (exception& e) {
          
          cout << "Exception thrown. \n" << e.what() << endl;
      }
    }

whose output is:

[ADD OUTPUT HERE]

A regex replace callback
------------------------

``sregex_iterator`` can be used together with the methods of ``smatch`` to do conditional replacement using a callback.

This code example capitalizes all titles\ |ndash|\ mr, ms, mrs, and dr\ |ndash|\ as well as ensures that all sentences begin with an uppercase word. It does his by initially putting all
the text in lowercase\ |ndash|\ since it may be all capitalized. Then, it uses a lamda function that takes a ``const smatch&`` to capitalize the proper letter. It relies on the smatch ``position(int)``
method to do so: 

.. code-block:: cpp

    // Initially ensure all text is all lowercase.
    for_each(s.begin(), s.end(), [&](char& c) { \
                                     c = tolower(c, locale()); }\
    ); 
    
    // Capitalize titles and the personal pronoun i
    regex re_titles{ R"(\b(?:dr)|(?:mr)|(?:ms)|(?:mrs)|(?:i)\b)" };
    
    sregex_iterator titles_iter(s.begin(), s.end(), re_titles);
    
    sregex_iterator titles_end;
    
    int index = 0;
    
    auto lambda_toupper = [&](const smatch& sm) {
    
         auto pos = sm.position(index);
         
         s[pos] = toupper(s[pos], std::locale());
    };
    
    for_each(titles_iter, titles_end, lambda_toupper);
    
    // Capitalize first word of sentences...
    
    // ...Do first character manually
    s[0] = toupper(s[0], locale());
    
    regex re_ucfirst{R"((?:\.|\?|!)\s+([a-z]))"};
    
    sregex_iterator ucfirst_iter(s.begin(), s.end(), re_ucfirst);
    
    sregex_iterator ucfirst_end;
    
    index = 1;
    
    // Do remaining sentences using regex and lambda_toupper function 
    for_each(ucfirst_iter, ucfirst_end, lambda_toupper);
    
    return;
