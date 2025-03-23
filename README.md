import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import Calendar from "react-calendar";
import 'react-calendar/dist/Calendar.css';

export default function DalantApp() {
  const [userList, setUserList] = useState([]);
  const [currentUser, setCurrentUser] = useState(null);
  const [loginId, setLoginId] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const [newUser, setNewUser] = useState({ name: "", password: "", role: "user" });
  const [selectedDate, setSelectedDate] = useState(new Date());
  const [showRecords, setShowRecords] = useState(false);
  const [showSignup, setShowSignup] = useState(false);

  useEffect(() => {
    const storedUsers = JSON.parse(localStorage.getItem("users")) || [];
    setUserList(storedUsers);
  }, []);

  useEffect(() => {
    localStorage.setItem("users", JSON.stringify(userList));
  }, [userList]);

  const handleLogin = () => {
    const user = userList.find((u) => u.name === loginId && u.password === password);
    if (user) {
      setCurrentUser(user);
      setError("");
    } else {
      setError("로그인 실패: 사용자 이름 또는 비밀번호가 올바르지 않습니다.");
    }
  };

  const handleSignup = () => {
    if (!newUser.name || !newUser.password) {
      setError("회원가입 실패: 모든 필드를 입력하세요.");
      return;
    }
    if (userList.some((u) => u.name === newUser.name)) {
      setError("회원가입 실패: 이미 존재하는 이름입니다.");
      return;
    }
    const updatedUsers = [...userList, { ...newUser, id: userList.length + 1, attendance: [], points: 0 }];
    setUserList(updatedUsers);
    setNewUser({ name: "", password: "", role: "user" });
    setError("");
  };

  const handleAttendance = (id) => {
    setUserList((prev) =>
      prev.map((user) =>
        user.id === id ? { ...user, attendance: [...user.attendance, new Date().toISOString().split('T')[0]], points: user.points + 5 } : user
      )
    );
  };

  const handleGivePoints = (id, amount) => {
    setUserList((prev) =>
      prev.map((user) =>
        user.id === id ? { ...user, points: user.points + amount, attendance: [...user.attendance, new Date().toISOString().split('T')[0]] } : user
      )
    );
  };

  return (
    <div className="p-4 grid gap-4 max-w-lg mx-auto">
      <h1 className="text-xl font-bold">달란트 시스템</h1>
      {!currentUser ? (
        <div>
          <h2 className="text-lg font-semibold">로그인</h2>
          <Input type="text" placeholder="이름 입력" value={loginId} onChange={(e) => setLoginId(e.target.value)} />
          <Input type="password" placeholder="비밀번호 입력" value={password} onChange={(e) => setPassword(e.target.value)} />
          <Button onClick={handleLogin}>로그인</Button>
          {error && <p className="text-red-500">{error}</p>}
          <Button onClick={() => setShowSignup(!showSignup)} className="mt-4">
            {showSignup ? "회원가입 숨기기" : "회원가입 하기"}
          </Button>
          {showSignup && (
            <div>
              <h2 className="text-lg font-semibold mt-4">회원가입</h2>
              <Input type="text" placeholder="이름 입력" value={newUser.name} onChange={(e) => setNewUser({ ...newUser, name: e.target.value })} />
              <Input type="password" placeholder="비밀번호 입력" value={newUser.password} onChange={(e) => setNewUser({ ...newUser, password: e.target.value })} />
              <select onChange={(e) => setNewUser({ ...newUser, role: e.target.value })}>
                <option value="user">일반 사용자</option>
                <option value="admin">관리자</option>
              </select>
              <Button onClick={handleSignup}>회원가입</Button>
            </div>
          )}
        </div>
      ) : (
        <div>
          <Button onClick={() => setShowRecords(!showRecords)} className="mb-2">
            {showRecords ? "출석 및 달란트 지급 기록 숨기기" : "출석 및 달란트 지급 기록 보기"}
          </Button>
          {showRecords && (
            <>
              <h2 className="text-lg font-semibold">출석 및 달란트 지급 기록</h2>
              <Calendar value={selectedDate} onChange={setSelectedDate} tileClassName={({ date }) => {
                const dateString = date.toISOString().split('T')[0];
                return currentUser.attendance.includes(dateString) ? "bg-green-200" : "";
              }} />
              <h2 className="text-lg font-semibold">사용자 목록</h2>
              {userList.map((user) => (
                <Card key={user.id} className="p-2">
                  <CardContent>
                    <p className="font-semibold">{user.name} ({user.role === "admin" ? "관리자" : "일반"})</p>
                    <p>달란트: {user.points}</p>
                    {currentUser.id === user.id && user.role === "user" && (
                      <Button onClick={() => handleAttendance(user.id)} disabled={user.attendance.includes(new Date().toISOString().split('T')[0])}>
                        출석 체크
                      </Button>
                    )}
                    {currentUser.role === "admin" && user.role === "user" && (
                      <Button onClick={() => handleGivePoints(user.id, 10)} className="ml-2">
                        +10 달란트 지급
                      </Button>
                    )}
                  </CardContent>
                </Card>
              ))}
            </>
          )}
        </div>
      )}
    </div>
  );
}
# -
