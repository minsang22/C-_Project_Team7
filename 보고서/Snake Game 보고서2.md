

- 아직 못채운부분 : 각 단계별 실행화면을 넣어야 함, 프로젝트 제작 기간,

#### 1. 프로젝트 제목

Snake Game

#### 2. 프로젝트 깃허브 링크

https://github.com/minsang22/CPP_Project_Team7

#### 3. 프로젝트 주제

Ncurses 라이브러리를 사용하여 C++로 Snake Game을 구현

#### 4. 프로젝트 팀원

- 최혁태 (조장)
- 윤민상
- 김하연

#### 5. 프로젝트 제작 기간

5월 ??일 ~ 6월 ??일

#### 6. 프로젝트 제작 일정

달력내용 ㅇㅇ

#### 7. 세부 구현 내용

- 1단계 - Snake Map 구현

  - 소스 코드 설명 (주석 참고)

  ```c++
  #include <ncurses.h>
  
  int main()
  {
      //윈도우 세개 생성
      WINDOW *win1;
      WINDOW *win2;
      WINDOW *win3;
      initscr();
      
      start_color();
      init_pair(1, COLOR_WHITE, COLOR_BLUE);      //글씨는 흰색, 배경은 파란색
      border('*','*','*','*','*','*','*','*'); mvprintw(1,1,"A default window"); refresh();   //default window 테두리 *
      getch();
  
      //win1은 뱀이 움직이는 공간
      win1 = newwin(22, 50, 3, 3);
      wbkgd(win1, COLOR_PAIR(1));
      wattron(win1, COLOR_PAIR(1));
      mvwprintw(win1, 1, 1, "Space for snake"); 
      wborder(win1, '@','@','@','@','@','@','@','@'); wrefresh(win1);
      getch();
  
  
      //win2는 점수를 표시하는 공간
      win2 = newwin(10, 40, 3, 56);
      wbkgd(win2, COLOR_PAIR(1));
      wattron(win2, COLOR_PAIR(1));
      mvwprintw(win2, 1, 2, "Score Board"); 
      mvwprintw(win2, 4, 2, "B: Current Length / Max Length");
      mvwprintw(win2, 5, 2, "+: Number of Growth Items"); 
      mvwprintw(win2, 6, 2, "-: Number of Poison Items"); 
      mvwprintw(win2, 7, 2, "G: Number of gate used"); 
      wborder(win2, '@','@','@','@','@','@','@','@'); wrefresh(win2);
      getch();
  
  
      //win3은 미션을 표시하는 공간
      win3 = newwin(10, 40, 15, 56);
      wbkgd(win3, COLOR_PAIR(1));
      wattron(win3, COLOR_PAIR(1));
      mvwprintw(win3, 1, 1, "Mission Board"); 
      mvwprintw(win3, 4, 1, "B: 10  ( )"); 
      mvwprintw(win3, 5, 1, "+: 5   ( )"); 
      mvwprintw(win3, 6, 1, "-: 2   ( )"); 
      mvwprintw(win3, 7, 1, "G: 1   (v)"); 
      wborder(win3, '@','@','@','@','@','@','@','@'); wrefresh(win3);
      getch();
  
      delwin(win1);
      endwin();
  return 0; }
  ```

  우선 Snake Map, Score Board, Mission Board로 사용할 Window 객체 3개를 선언해주었다. 그리고 각각의 Window에 wborder 함수를 이용해 테두리를 설정해주고, mwprintw 함수를 이용해 내용을 출력해주었다.

  

  - 구현 내용

    

    Snake가 움직일 기본 Map과 동시에 Score, Mission Board도 표시해주도록 구현하였다.

  - 실행화면

    

- 2단계 - Snake와 Snake의 조작 구현

  - 소스 코드 설명 (주석 참고)

  ```c++
  class Vector //Snake를 움직일 때 사용하기 위한 Vector 클래스
  {
  	public:
  		Vector() : x(0), y(0){}
  		Vector(int x, int y) {setX(x); setY(y);}
  
  		void setX(int x){this->x = x;} //x좌표를 설정해주기 위한 세터 메소드
  		int getX() const {return this->x;} //x좌표를 받아오기 위한 게터 메소드
  		void setY(int y){this->y = y;} //y좌표를 설정해주기 위한 세터 메소드
  		int getY() const {return this->y;} //y좌표를 받아오기 위한 게터 메소드
      
      	//Snake가 움직일 때 좌표값 설정을 위한 연산자 오버로딩 
  		Vector& operator=(const Vector& vec)
  		{setX(vec.getX()); setY(vec.getY()); return *this;}
  		Vector& operator+=(const Vector& vec)
  		{setX(getX()+vec.getX()); setY(getY()+vec.getY()); return *this;}
  		Vector operator+(const Vector& vec)
  		{Vector v(getX(),getY()); return v+=vec; }
  		Vector& operator-=(const Vector& vec)
  		{setX(getX()-vec.getX()); setY(getY()-vec.getY()); return *this;}
  		Vector operator-(const Vector& vec)
  		{Vector v(getX(),getY()); return v-=vec;}
  		bool operator==(const Vector& vec)
  		{return getX()==vec.getX() && getY()==vec.getY();}
  		bool operator!=(const Vector& vec)
  		{return !operator==(vec);}
  			
  	private:
  		int x,y;
  };
  ```

  우선 Vector 클래스를 통해 맵의 x, y좌표를 사용하도록 하였다. 세터 메소드와 게터 메소드를 통해 맵의 x, y 좌표를 얻어오고 설정해줄 수 있도록 하였고, 연산자 오버로딩을 통해 좌표 사이의 연산을 정의해주었다.

  

  ```c++
  class Snake //Snake를 구현하기 위한 클래스
  {
      			//중략
      
  	void getApple()
  	{
  		int x = rand()%width; 
  		int y = rand()%height;
          //x, y 랜덤 난수 생성
  		apple = Vector(x,y); //apple 아이템 좌표 지정
  		for(unsigned int i=0;i<body.size();++i)
  			if(apple==body[i])getApple();
              //apple 아이템이 Snake와 겹치지 않는지 검사해주기 위함
  	}
  ```

  그 다음으론 Snake 클래스 안에 다양한 멤버 변수와 메소드를 구현해 게임을 위한 다양한 기능들을 구현하였다. 우선 getApple 함수는 Apple 아이템이 랜덤으로 생성될 좌표를 지정해주기 위한 함수이다. rand 함수를 사용해 지정된 좌표 범위 내에서 x, y를 랜덤 난수로 지정해주고 Vector를 통해 apple 아이템의 좌표를 지정해준다. 그리고 apple 아이템이 만약 현재 Snake가 있는 좌표에 생성되는지 검사해주고 만약 겹친다면 새로운 좌표를 지정해준다.

  

  ```c++
  void setDirection(int d)
  	{
  		switch(d)
  		{	//키보드로 입력받는 값에 따라 이동할 좌표값 설정
  			case 0:direction = Vector(0, -1); break;
  			case 1:direction = Vector(1, 0); break;
  			case 2:direction = Vector(0, 1); break;
  			case 3:direction = Vector(-1, 0); break;
  		}
  	}
  ```

  setDirection 함수는 Snake의 이동 방향을 설정해주기 위한 함수이다. switch문을 통해 키보드로 입력받는 값에 따라 이동할 좌표값을 Vector로 설정해준다.

  

  ```c++
  void makeMove() //Snake를 실제로 움직이게 해주는 함수
  	{
  		exit = false;
  		if(body[0].getX()<0 || body[0].getX()>=width)
  			exit = true;
  		else if(body[0].getX()==0)
  		{	if(direction==Vector(-1,0))
  				exit = true;
  		}
  		else if(body[0].getX()==(width-1))
  		{	if(direction==Vector(1,0))
  				exit = true;
  		}
  			
  		if(body[0].getY()<0 || body[0].getY()>=height)
  			exit = true;
  		else if(body[0].getY()==0)
  		{
  			if(direction==Vector(0,-1))
  				exit = true;
  		}
  		else if(body[0].getY()==(height-1))
  		{
  			if(direction==Vector(0,1))
  				exit = true;
  		}
  
  		if(!exit)body[0]+=direction;
  
  		for(unsigned int i = 1; i<body.size(); ++i)
  			if(body[i]==body[0]){exit = true; break;}
  	}
    
  	void bodyMove() 
  	{
  		for(unsigned int i=body.size()-1; i>0; --i)
  			snakeSwap(i);
  	}
  ```

  makeMove, bodyMove 함수는 실제로 Snake를 움직이게 해주는 함수이다. exit 변수가 true가 된다면 Snake가 Map 바깥으로 탈출했다는 의미이므로 게임이 끝나게 된다.

  

  ```c++
  void checkForApple()
  	{
  		if(body[0]==apple) //Snake의 head부분이 apple 아이템을 만났다면
  		{
  			points++; //포인트 증가
  			if(points>best)best = points; //베스트 스코어라면 갱신해줌 
  			getApple(); //새로운 Apple 아이템 생성
  			snakeSwap(body.size()); //Snake size 증가
  		}
  	}
  		
  ```

  checkForApple 함수는 main문에서 반복문을 통해 계속 호출하며 Snake가 Apple 아이템을 먹었는지 체크해줄 함수이다. Snake의 머리 부분인 body[0]이 apple의 좌표와 같다면 Snake가 아이템을 먹은 것이기 때문에 새로운 Apple 아이템을 생성해주는 동시에 Snake의 길이를 1 증가시켜준다. 

  

  ```c++
  void checkPoints()
  	{
  		switch(points){
  			case 3: level = 2; speed = 98; break;
  			case 5: level = 3; speed = 85; break;
  			case 7: level = 4; speed = 80; break;
  			case 9: level = 5; speed = 60; break;
  			case 50: level = 6; speed = 80; break;
  			case 60: level = 7; speed = 65; break;
  			case 70: level = 8; speed = 50; break;
  			case 80: level = 9; speed = 40; break;
  			case 90: level = 10; speed = 30; break;
  			case 100: level = 999; speed = 15; break;
  		}
  	}
  ```

  checkPoints 함수는 일정한 포인트에 따라 Snake의 속도를 바꿔주는 함수이다. 일정한 속도로만 게임이 진행된다면 질리는 것을 방지하기 위해 points에 따라 speed에 변화를 주어 좀 더 재미를 부가시켜 주었다.

  

  ```c++
  void snakeSwap(uint i)
  	{	
  		if(i >= body.size())
  			body.push_back(body.back());
  		else
  			body[i] = body[i - 1];
  	}
  ```

  snakeSwap은 Snake의 body가 head를 따라 움직일 수 있도록 만들어준 함수이다. 매개변수로 받는 i 만큼의 body가 head를 따라간다.

  

  ```c++
  int main()
  {
  	initscr();
  	noecho();
  	cbreak();
  	int x,y;
  	getmaxyx(stdscr, y, x);
  	int best = getBest();
  	//render frame
  	WINDOW *win = newwin(y-3, x, 1, 0); //height, width, startY, startX
  	box(win, 0, 0);
  	nodelay(win, TRUE);
  	keypad(win, TRUE);
  
  	WINDOW *score = newwin(1, COLS, 0,0);
  
  	do
  	{
  		Snake snake(y-5, x-2, best);
  		refresh();
  		wrefresh(win);
  		printScore(score, 0, 1, best);
  		while(!snake.getExit())
  		{
  			char *tbl = snake.getTable();
  			//process data
  			//draw
  			draw(win, snake, tbl, snake.getHeight(), snake.getWidth());
  			
  			printScore(score, snake.getPoints(), snake.getLevel(), snake.getBest());
  			
  			int input = wgetch(win);
  			proccesInput(win, snake, input);
  			if(snake.getExit())break;
  
  			snake.bodyMove();
  			snake.makeMove();
  			snake.checkForApple();
  			snake.checkPoints();
  
  			//-----[ SLEEP ]-------
  			#ifdef WIN32
  			Sleep(snake.getSpeed());
  			#else
  			usleep(snake.getSpeed() * 1000);
  			#endif
  			//---------------------
  		}	
  		if(snake.getBest()>best){writeBest(snake.getBest()); best = snake.getBest();}
  	}while(writeEndAndGetInput());
  	
  	delwin(score);
  	delwin(win);
  	endwin();
  }
  ```

  main문은 다음과 같이 구성하였다. do - while문을 이용해 do문에서 우선 기본적인 게임의 설정을 해주고, while 루프가 돌아가며 게임이 진행되도록 구현해주었다. 1단계에서 구현한 코드를 바탕으로 여러 기능들을 더해 구현하였다.

  

  ```c++
  std::string getFile() //아래 두 함수에 사용할 파일을 불러오기 위함
  {
  	#ifdef WIN32
  	std::string home = "%appdata%/.md.snake\0";
  	#else
  	const char *homedir;
  
  	if ((homedir = getenv("HOME")) == NULL) 
  	    homedir = getpwuid(getuid())->pw_dir;
  	
  	std::string home = std::string(homedir)+"/.md.snake\0";
  	#endif
  	return home;
  }
  
  int getBest() //저장한 best 스코어를 받아오기 위함
  {
  	ifstream fp (getFile().c_str());
  	int best = 0;
  	std::string content;
  	if(fp.is_open() && fp.good())
  	{
  		getline(fp, content); 
  		best = strtol(content.c_str(), NULL, 10);
  		fp.close();
  	}
  	return best;
  }
  
  void writeBest(int best) //best 스코어를 저장하기 위함
  {
  	ofstream fp (getFile().c_str());
  	if(fp.is_open())
  	{
  		fp << best;
  		fp.close();
  	}
  }
  ```

  이 세개의 함수들은 Snake Game의 베스트 스코어를 저장하기 위한 파일을 불러오고, 파일에서 베스트 스코어를 받아오는 동시에 저장해주는 역할을 한다.

  

  ```c++
  void proccesInput(WINDOW* win, Snake& snake, int input)
  {
  	int d = snake.getDirection();
  	switch(input)
  	{
  		case KEY_UP:
  			if(d!=0 && !(d==2 && snake.getPoints() > 0))
  			snake.setDirection(0);
  			break;
  		case KEY_DOWN:
  			if(d!=2 && !(d==0 && snake.getPoints() > 0))
  			snake.setDirection(2);
  			break;
  		case KEY_LEFT:
  			if(d!=3 && !(d==1 && snake.getPoints() > 0))
  			snake.setDirection(3);
  			break;
  		case KEY_RIGHT:
  			if(d!=1 && !(d==3 && snake.getPoints() > 0))
  			snake.setDirection(1);
  			break;
  		case 'Q':
  		case 'q':
  			snake.setExit(true);
  			break;
  		case 'P':
  		case 'p':
  			char c;
  			wattron(win, A_BOLD);
  			mvwprintw(win, snake.getHeight()/2, snake.getWidth()/2, "PAUSE");
  			wattroff(win, A_BOLD);
  			nodelay(win, FALSE);
  			do{
  				c = wgetch(win);
  			}while(c!='p' && c!='P');
  			nodelay(win, TRUE);
  			break;
  	}
  ```

  processInput 함수는 키보드로 받는 입력을 Snake Game에 맞게 전달해주기 위한 함수이다. 기본적으로 방향키를 통해 Snake를 움직일 수 있도록 해주었고, p 버튼을 누를시 게임을 일시중지 하는 기능도 추가하였다. 

  

  - 구현 내용

    Snake가 움직이는 것을 구현함과 동시에 Growth Item까지 2단계에서 미리 구현해보았다. Grwoth Item의 이름은 따로 Apple로 설정해 주었고, 게임의 best 스코어를 저장함과 동시에 받아오는 것도 구현해 주었고, 게임의 재미를 위해 포인트에 따라 속도를 변경해 주는 기능을 추가하였다. 또, p 버튼을 통해 게임을 일시중지 하는 기능도 추가하였다. 

    

  - 실행화면

    

- 3단계 - Growth / Poison Item 구현

  - 소스 코드 설명 (주석 참고) 

  ```c++
  void dropPoison(int map[30][60])  // poison 위치 설정
  		{	
  			startPT = time(NULL);
  			
       	int x,y;
  			x = rand()%width;
  			y = rand()%height;
  			poison = Vector(x,y);
  			for(unsigned int i=0;i<body.size();++i)
  				if(poison==body[i])dropPoison(map);
              if(map[y][x]!=0) dropPoison(map);
  		}
  
  void checkForPoison() // 길이 줄이고 포인트 감소
  		{	
  
  			nowPT = time(NULL);
  			
  			if(nowPT - startPT > itemT){
  				dropPoison(map);
  			}
  
  			if(body[0]==poison)
  			{	countP++; //수정
  				points--;
  				dropPoison(map);
  				reduceBodySize(body.size());
  			}
  		}
  
  void reduceBodySize(uint i)
  		{
  			if(i >= body.size())
  				body.pop_back();
  			else
  				body[i] = body[i - 1];
  		}
  ```

  2단계의 코드에 새롭게 추가한 부분이다. 아이템을 5초간 획득하지 못할시엔 아이템이 재생성되는 기능을 추가해주는 과정에서 2단계에서 Apple 아이템을 구현해 준 것에 부족한 부분을 발견해 다른 방식으로 Poison 코드를 작성하고 Apple 아이템 코드도 Poison과 같은 방식으로 변경해주었다. 우선 아이템을 랜덤 좌표를 생성에 맵에 아이템을 생성해주는 dropPoison 함수를 만들고, 이 함수가 호출될 때마다 현재 시간으로 startPT를 갱신해준다. 현재 시간을 갱신해줄때는 ctime 라이브러리의 time(NULL) 함수를 이용하였다. 그리고 main의 while문에서 아이템을 획득했는지 검사해줄 checkForPoison 함수를 만들어주었다. 반복문에서 호출되며 계속 nowPT를 현재 시간으로 갱신해준다. 만약 nowPT - startPT가 itemT( == 5초)보다 커질 시엔 아이템을 재생성해주는 식으로 해주었다. 그리고 reduceBodySize함수를 새로 만들어주어 Snake의 Size를 조정해주도록 하였다.

  

  - 구현 내용

    Poison 아이템을 구현해주었고, 2단계에서 구현했던 Apple 아이템과 Poison 아이템에 5초간 아이템을 획득하지 못할시엔 아이템이 재생성되는 기능을 추가해주었다.

    

  - 실행화면

    

- 4단계 - Gate 구현

  - 소스 코드 설명 (주석 참고)

  ```c++
  void checkWall(int map[30][60]) 
  		{
  
              countW =0;
  			for(int i=0; i<30; i++){
                  for(int j=0; j<60; j++)
                      if(map[i][j]==1){
                      wall1[countW++] = Vector(j,i);            
                  }
              }    
  		}
  
  void getGate(Vector wall[])
          {
              int A = rand()%countW;
              int B = rand()%countW;
              if(B == A) getGate(wall1);
              gate[0] = wall1[A];
              gate[1] = wall1[B];
          }
  
  void checkGate() // 몸이 다 통과할때까지 머리 방향 설정
          {
              if(body[0] == gate[0]){
                  body[0] = gate[1];
                  countG++;
                  setDirection(gateDirection(gate[1]));
              }
  
              else if(body[0] == gate[1]){
  				countG++;
                  body[0] = gate[0];
                  setDirection(gateDirection(gate[0]));
              }
                 
          } 
  
  void changeGate() 
  {
  			Vector A = direction;
  			for(int i=0; i<body.size(); i++){
  				A = A + direction;
  
  			}
              if(body[0] == gate[0]+A){
  				getGate(wall1);
              }
  
              else if(body[0] == gate[1]+A){
  				getGate(wall1);
              }
                  
          }
  ```

  우선 벽의 위치를 저장해주기 위해 벽이 생성된는 위치를 Vector 배열로 생성해주는 checkWall 함수를 만들어주었다. for 반복문을 통해 맵을 검사해주며 값이 1인 좌표(벽을 1로, 모서리를 2로 표현함)들을 Vector 배열로 저장해주었다. 그 후, getGate 함수에서 랜덤 난수 생성을 통해 게이트의 양쪽 입구를 설정해주었다. 그리고 checkGate 함수에서는 몸이 다 통과할 때 까지의 몸의 방향을 설정해주어 한쪽 게이트를 통과하면 다른쪽 게이트로 나올 수 있도록 해주었다. changeGate 함수에서는 몸이 게이트를 온전히 다 통과해야 다른 위치에 게이트를 재생성해주도록 하여 게이트를 완성하였다.

  

  - 구현 내용

    Gate가 생성되는 것과 Gate를 올바르게 통과고 재생성되는 과정까지 구현하였다.

    

  - 실행 화면

    

- 5단계 - 점수 표시 화면 구현

  - 소스 코드 설명 (주석 참고)

  ```c++
  void printScore(WINDOW* w, int score, int level, int sz)
  {
  	werase(w);
  	wbkgd(w, COLOR_PAIR(1));
      wattron(w, COLOR_PAIR(1));
  	wborder(w, '|','|','-','-','+','+','+','+');
  	 mvwprintw(w, 1, 2, "Score Board"); 
      mvwprintw(w, 4, 2, "B: Current Length / Max Length %d/%d",sz,missionB);
      mvwprintw(w, 6, 2, "+: Number of Growth Items %d",countA ); 
      mvwprintw(w, 8, 2, "-: Number of Poison Items %d",countP); 
      mvwprintw(w, 10, 2, "G: Number of gate used %d",countG); 
  	wrefresh(w);
  }
  
  void printMission(WINDOW* w, int score, int level)
  {
  	werase(w);
  	wbkgd(w, COLOR_PAIR(1));
      wattron(w, COLOR_PAIR(1));
  	wborder(w, '|','|','-','-','+','+','+','+');
      mvwprintw(w, 1, 2, "Mission Board"); 
      mvwprintw(w, 4, 2, "B: %d   (%c)",missionB,comB); 
      mvwprintw(w, 5, 2, "+: %d   (%c)",missionA,comA); 
      mvwprintw(w, 6, 2, "-: %d   (%c)",missionP,comP); 
      mvwprintw(w, 7, 2, "G: %d   (%c)",missionG,comG); 
  	wrefresh(w);
  }
  ```

  ncurses 라이브러리의 mvwprintw를 사용하여 1단계에서 썻던 방식처럼 WINDOW 객체에 점수와 미션 진행 상황을 출력해주었다.

  

  - 구현 내용

    Map 오른쪽에 Score Board와 Mission Board를 출력해주었다.

    

  - 실행 화면

    

- 마무리 단계 - 기능 추가 / 코드 수정

  - 소스 코드 설명 (주석 참고)

  ```c++
  void mapChange(int next_map[30][60]){ 
              for(int i=0; i<30; i++){
                  for(int j=0; j<60; j++){
                      map[i][j] = next_map[i][j];
                  }
              }
              checkWall(next_map);
  			
          }
  ```

  주어진 미션을 충족해 다음 단계로 넘어갈 때에 맵을 바꿔주기 위한 함수이다. for 반복문을 돌며 map에 next_map을 복사해준 후, checkWall 함수를 호출해 벽의 위치를 저장하는 Vector 배열까지 만들어준다.

  

  ```c++
  void changeBody(){
  		for(int i=0; i<body.size()-3; i++){
  			reduceBodySize(body.size());
  		}	
  	}
  ```

  다음 단계 맵으로 넘어갈 때에 Snake의 size를 다시 3으로 줄여주기 위해 만들어준 함수이다. for 반복문을 돌며 Snake의 size가 3이 될 때까지 size를 감소시켜준다.

  

  ```c++
  void getSize(){ 
  			if(body.size()<3){exit = true;}
  		}
  ```

  게임의 기본 조건인 Snake의 size가 3보다 작아질 시 게임이 끝나게 되는 조건을 getSize 함수로 구현해주었다.

  

  ```c++
  void mission(){
  			if(missionA <= countA){
  				comA = 'v';
  			}
  			else comA = ' ';
  			if(missionP <= countP){
  				
  				comP = 'v';
  			}
  			else comP = ' ';
  			if(missionG <= countG){
  		
  				comG = 'v';
  			}
  			else comG = ' ';
  			if(missionB <= body.size()){
  		
  				comB = 'v';
  			}
  			else comB = ' ';
  		}
  
  		int missionComplete(){
  			if(comP == 'v' && comA == 'v' && comG == 'v' && comB == 'v' ) {
  				level++;
  				return level;
  			}
  			return level;
  		}
  ```

  게임 플레이 도중 맵의 오른쪽에 출력되는 Mission Board의 가독성을 높이기 위해 각 미션 조건 충족시 v 표시를 해주는 기능을 mission 함수에 추가해주었다. 그리고 모든 미션이 충족되면 게임이 다음 단계로 넘어갈 수 있도록 해주는 missionComplete 함수도 만들어주었다.

  

  ```c++
  void checkPoints() 
  		{
  			switch(missionComplete()){
  				case 2: {
  					missionA =2;
  					missionB =4;
  					missionP =2;
  					missionG =2;
  					mapChange(map2);
  					break;}
  				case 3: {
  					missionA =3;
  					missionB =5;
  					missionP =3;
  					missionG =3;
  					mapChange(map3); break;}
  				case 4: {
  					missionA =4;
  					missionB =6;
  					missionP =4;
  					missionG =4;
  					mapChange(map4); break;}
  				case 5: {
  					missionA =5;
  					missionB =7;
  					missionP =5;
  					missionG =5;
  					mapChange(map5); break;}				
  			}
  		}
  ```

  미션 완료시 다음 단계로 넘어갈 때 mapChange 함수를 호출해 맵을 바꿔주고, 새로운 미션을 부여해주는 기능을 하는 checkPoints 함수를 구현해주었다.

  

  ```c++
  	mvwprintw(endwin, 14, 40, "GAME OVER");
  	mvwprintw(endwin, 15, 40, "Game Score : %d", score);
  	mvwprintw(endwin, 16, 40, "Play Time : %d second", overGT - startGT);
  	mvwprintw(endwin, 17, 40, "Total Score : %d", score + 1000 - (overGT - startGT));
  	mvwprintw(endwin, 18, 40, "Press [q] to quit.");
  ```

  main문에서 게임이 끝났을 때 보여주는 Window 객체 중 endwin에 mvwprintw 함수를 통해 게임의 점수를 출력해 주도록 하였다. 게임 점수는 Apple 아이템을 먹은 횟수 + Snake의 현재 길이 - Poison 아이템을 먹은 횟수 + Gate를 사용한 횟수 * 2로 설정해 주었다. 그리고 게임이             끝난 시간 - 게임을 시작한 시간으로 플레이 시간을 초 단위로 계산해주어 출력해주고, 총 점수는 게임 점수 + (1000 - 플레이 시간)으로 설정해 주었다.

  

- 삭제한 기능

  2단계에서 게임의 재미를 위해 획득한 게임 점수에 따라 속도를 변경해주는 기능을 추가했었는데, 게임 단계가 바뀌며 맵도 바뀌도록 해주며 상위 단계에선 속도가 너무 빠르면 게임의 난이도가 너무 어려워져 기능을 삭제하였다.

#### 8. 소감(이거 쓸까? 말까 ? ? ?)

- 최혁태
- 윤민상
- 김하연