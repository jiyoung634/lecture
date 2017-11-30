# lecture

package com.grade004;

import java.util.Scanner;

// 메인 메뉴 액션 클래스
public class Main {

	public static void main(String[] args) {
	
		Scanner sc = new Scanner(System.in);
		boolean run = true;
		ScoreService service = new ScoreService();
		
		// 메인 메뉴 액션
		while(run) {
			System.out.println("1. 성적 정보 입력  2. 성적 정보 출력");
			System.out.println("선택(1~2, 0 quit)?");
			int answer = sc.nextInt();
			sc.nextLine();
			
			switch (answer) {
			case 1: service.scoreAdd(sc); break;
			case 2: service.scoreList(); break;
			case 0: run = false; break;			
			}
		}
		sc.close();
		System.out.println("프로그램이 종료되었습니다");
	}

}

//---------------------------------------------------------------------

package com.grade004;

// 자료형 클래스
// 화면 설계를 토대로 항목 결정 
// 번호(읽기 전용), 이름, 과목1, 과목2, 과목3, 총점(읽기 전용), 평균(읽기 전용), 석차 
// 읽기 전용 멤버는 getter만 구성한다.

//
public class Score {
	private String number, name;
	private int subject1, subject2, subject3, sum;
	private double average;
	
	// 석차 정보 저장용 필드 추가
	// 주의) 석차 초기값은 1로 설정해야 한다.
	private int rank=1;

	
	
	// 생성자
	public Score() {
	}

	public Score(String number, String name, int subject1, int subject2, int subject3) {
		this.number = number;
		this.name = name;
		this.subject1 = subject1;
		this.subject2 = subject2;
		this.subject3 = subject3;
	}

	// getter

	public String getNumber() {
		return number;
	}

	public String getName() {
		return name;
	}

	public int getSubject1() {
		return subject1;
	}

	public int getSubject2() {
		return subject2;
	}

	public int getSubject3() {
		return subject3;
	}
	
	public int getRank() {
		return rank;
	}
	
	
	// 총점 계산용 메소드
	public int getSum() {
		return this.subject1 + this.subject2 + this.subject3;
	}
	
	// 평균 계산용 메소드
	public double getAverage() {
		return this.getSum() /3.0;
	}

	// setter

	public void setNumber(String number) {
		this.number = number;
	}

	public void setName(String name) {
		this.name = name;
	}

	public void setSubject1(int subject1) {
		this.subject1 = subject1;
	}

	public void setSubject2(int subject2) {
		this.subject2 = subject2;
	}

	public void setSubject3(int subject3) {
		this.subject3 = subject3;
	}

	public void setRank(int rank) {		
		this.rank = rank;
	}

	// 성적 정보 출력용 메소드 추가
	public String scoreInfo() {
		return String.format("%-5s %-5s %5d %5d %5d %6d %6.1f %5d %n", this.number, this.name, this.subject1,
				this.subject2, this.subject3, this.getSum(), this.getAverage(), this.rank);
	}

}
//------------------------------------------------------------------------

package com.grade004;

import java.util.Arrays;

public class ScoreDAO {
	//(배열)저장소 생성 및 관리	
	private Score[] scores = new Score[5];
	
	//(배열)저장소 인덱스 관리 변수	
	private int idx;
	
	// 성적 정보 입력 메소드
	// 주의) 중복된 번호 사용시 입력 실패할 수 있다.
	public int scoreAdd(String number, String name, int subject1, int subject2, int subject3) {
		int result = 0; // 입력 실패의 경우

		// 중복된 번호 검사 과정 추가 -> private 메소드
		if (this.numberCheck(number)) {

			// 객체 생성 및 (배열)저장소에 저장
			Score s = new Score(number, name, subject1, subject2, subject3);
			scores[idx] = s;
			// 인덱스 증가
			++idx;

			// (배열) 저장소 자동 증가 과정 추가
			// -> 큰 배열 준비하고, 기존 배열 요소 복사, 참조주소 연결
			if (this.idx % 5 == 0) {
				this.scores = Arrays.copyOf(this.scores, this.scores.length + 5);
			}
			
			result = 1;	// 입력 성공의 경우
		}
		return result;		
	}
	
	
	// 성적 정보 출력 메소드
	public String scoreList() {
		
		// 석차 계산 메소드 호출
		this.rank();
		StringBuilder list = new StringBuilder();
		
		list.append( String.format("총: %d건%n", idx));
		list.append( String.format("--------------------------------------------------%n"));
		list.append( String.format("번호  이름   과목1 과목2 과목3  총점   평균  석차%n"));
		list.append( String.format("--------------------------------------------------%n"));
		
		for(Score temp : scores) {
			if(temp == null) continue;
			list.append(temp.scoreInfo());
		}
		
		return list.toString();
	}
	
	
	// 석차 계산 메소드 -> private 메소드로 구성
	private void rank() {
		
		// 석차 알고리즘
		// 1. 배열에 성적 정보를 여러개 저장
		// 2. 모든 석차 정보는 초기값 1로 설정
		// 3. 기준 성적과 나머지 성적들을 비교해서 기준 성적보다 나머지 성적이 큰 경우가 있다면기준 석차의 현재값을 +1 연산.
		for(int i=0; i<this.scores.length; ++i) {
			Score s1 = this.scores[i];
			if(s1==null) continue;
			
			for(int j=0; j<this.scores.length; ++j) {
				Score s2 = this.scores[j];
				if(s2==null) continue;
				
				// 기준 성적보다 나머지 성적이 큰 경우
				if(s1.getSum()<s2.getSum()) {
					// 현재 석차 +1 액션
					s1.setRank(s1.getRank()+1);
				}
			}
		}
	}
	
	
	// 번호 중복 검사 메소드 -> private 메소드
	private boolean numberCheck(String number) {
		boolean result = true; 
		for(Score temp : scores) {
			if(temp==null) continue;
			if(number.equals(temp.getNumber())) {
				result = false;
				break;
			}
		}
		return result;
	}
	
	
}
//-----------------------------------------------------------------------

package com.grade004;

import java.util.Scanner;

// (콘솔) 서비스 관련 클래스
public class ScoreService {
	//(배열)저장소 생성 및 관리
	private ScoreDAO dao = new ScoreDAO();
	
	// 성적 정보 입력 메소드
	public void scoreAdd(Scanner sc) {
		System.out.println("--------------");
		System.out.println("성적 정보 입력");
		System.out.println("--------------");
		
		System.out.println("번호: ");
		String number = sc.nextLine();
		
		System.out.println("이름: ");
		String name = sc.nextLine();
		
		System.out.println("과목1: ");
		int subject1 = sc.nextInt();
		sc.nextLine();
		
		System.out.println("과목2: ");
		int subject2 = sc.nextInt();
		sc.nextLine();
		
		System.out.println("과목3: ");
		int subject3 = sc.nextInt();
		sc.nextLine();
	
		int result = this.dao.scoreAdd(number, name, subject1, subject2, subject3);
		if(result==1) {
			System.out.println("성적 정보 입력이 성공했습니다.");
		} else {
			System.out.println("성적 정보 입력이 실패했습니다. 중복된 번호 사용.");
		}
	}
	
	// 성적 정보 출력 메소드
	public void scoreList() {
		System.out.println("--------------");
		System.out.println("성적 정보 출력");
		System.out.println("--------------");
		
		System.out.println(this.dao.scoreList());
		
		System.out.println("--------------------------------------------------");
		
		
	}
	
}

